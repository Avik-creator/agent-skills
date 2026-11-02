# Controls

Read when implementing a control. Each rule is language-neutral. Go file, cookie, and CSRF helpers are in the go skill's `references/security.md`; use those instead of a second copy. Password parameters below were checked on 2026-11-02 against the OWASP Password Storage Cheat Sheet.

## Validation

Check every external value at the boundary, against an allowlist of shape, length, and allowed values. Code past the boundary uses the parsed value. Why: a check later is one that a new caller skips. Client-side checks do not count.

## Queries

Send data as parameters. Allowlist an identifier (table, column, order by) when it cannot be a parameter.

```sql
SELECT id FROM users WHERE id = ?
```

Why: concatenation makes the data into the query.

## Output encoding

Use the framework's escaping for the context the value lands in (HTML, attribute, URL). Why: a string that was safe in a query can still be markup in a page.

In the browser, plain text goes in with `textContent`, never `innerHTML`. HTML from a user goes through a sanitizer (such as DOMPurify) before `innerHTML`, `v-html` or `dangerouslySetInnerHTML`. A URL put into `href` or `src` has the scheme `http:` or `https:`, never `javascript:`. Why: each of these turns a string into running code.

## Authorization

On every request, check that this caller may do this action to this object, including the owner and the tenant. A missing object and an object the caller does not own should not tell the caller which one it was. Why: a valid session plus another user's id is enough to read their data.

## Passwords

Hash with Argon2id. If it is unavailable, use scrypt. Use bcrypt only where those two are unavailable. A unique salt is part of the hash; a library that generates it is fine. Why: a fast hash is cheap to guess.

Checked on 2026-11-02. OWASP's minimum Argon2id configurations (do not mix a memory value from one row with an iteration count from another):

| Memory (m) | Iterations (t) | Parallelism (p) |
|---|---|---|
| 47104 KiB (46 MiB) | 1 | 1 |
| 19456 KiB (19 MiB) | 2 | 1 |
| 12288 KiB (12 MiB) | 3 | 1 |
| 9216 KiB (9 MiB) | 4 | 1 |
| 7168 KiB (7 MiB) | 5 | 1 |

scrypt, same source, one of: N=2^17, r=8, p=1; N=2^16, r=8, p=2; N=2^15, r=8, p=3; N=2^14, r=8, p=5; N=2^13, r=8, p=10. bcrypt: work factor at least 10, and the input limit is 72 bytes. When a FIPS-140 module is required, OWASP says PBKDF2-HMAC-SHA-256 with at least 600,000 iterations instead.

## Session cookies

`HttpOnly`, `Secure`, and `SameSite=Lax` or `SameSite=Strict`. Leave `Domain` unset so the cookie stays on this host. Why: script can read a cookie that is not `HttpOnly`, and a `Domain` attribute sends it to every subdomain. `SameSite=None` sends it on cross-site requests.

## CSRF

For a cookie-authenticated state change (POST, PUT, PATCH, DELETE), set `SameSite` as above and check where the request came from: reject `Sec-Fetch-Site: cross-site`; otherwise the full `Origin` must match this site; with no `Origin`, the `Referer` host must match; with neither, or `Origin: null`, reject unless a CSRF token passes. Why: `SameSite` is missing on some clients, and a missing header is not proof of same origin. Checked on 2026-11-02 against the OWASP CSRF Prevention Cheat Sheet.

## Uploads

Allowlist the content types, cap the size before reading the whole body, and check the file's first bytes when the type matters. Store it under a name you generate, outside anything served as code. Why: the extension and the declared type are the uploader's claim.

## Redirects

A redirect target that comes from input is an allowlisted path or host, never a raw URL. Why: an open redirect lends your domain to a phishing link.

## Security headers

Send a content security policy that starts from the application's own origin, HSTS, `X-Content-Type-Options: nosniff`, a frame policy, and a referrer policy. Why: the browser will not guess these for you.

## CORS

List the allowed origins in configuration. A credentialed response (`Access-Control-Allow-Credentials: true`) uses that list, never `*`. Why: `*` with credentials lets any site call the API as the user.

## Errors

The client receives a fixed code and a fixed sentence. The driver error, the query, and the stack stay in the log. Why: the error text is a map of the system.

## SSRF

Any URL the caller influences (webhook, import, preview, redirect) is checked before the fetch: allowlisted scheme and host, every resolved address checked, private and reserved addresses rejected (loopback, link-local, private, unique-local, for both IP versions), redirects refused. Why: the server can be aimed at an address the caller cannot reach.

The check and the connection can resolve the name twice. For a high-risk fetch, connect to the address you already accepted. Say so in the residual risk when you do not.

## Paths named by data

A delete, move, or overwrite uses a path only after all three: symlinks are resolved, the result is inside an allowlisted root and is not the root itself, and ownership evidence was read before the call. On a no, log the rejected path and stop. A string prefix is the wrong check: a root of `/var/www/uploads` matches `/var/www/uploads2`. Compare with a relative path after the resolve. A relative result of `..`, a result that starts with `..` plus a separator, or an absolute result is outside. A child named `..cache` is inside; a check that rejects every string starting with `..` drops it.

Node's `path` module. Other languages need the same relative check after the symlink resolve.

```javascript
import { relative, isAbsolute, sep } from "node:path";

function insideRoot(root, target) {
  const rel = relative(root, target);
  if (rel === "" || rel === ".." || rel.startsWith(".." + sep) || isAbsolute(rel)) return false;
  return true;
}
```

Resolve `target` (follow symlinks) before calling `insideRoot`. In Go, open through `os.Root` instead; see the go skill.

Two limits. A marker file inside the tree is self-attestation: anyone who can write there can write the marker, so the expected owner comes from the authenticated caller, and the marker needs its own integrity (ownership of the file, or a MAC) before it authorizes anything. Resolving a path and then using the name races with a swap of a parent directory; where an untrusted process can do that, hold a file descriptor opened without following links, or make the tree immutable for the call.

## Rate limiting

Count in a store shared by every instance. An in-process counter becomes the limit times the number of instances, and a fresh process starts at zero. Take the limits from the product's own requirements; there is no universal number. Why: a limit that does not survive a second process is not a limit.

## Webhooks

Verify the signature over the raw body bytes before parsing or trusting anything in it, and refuse a missing or wrong one. Why: the body is whatever the sender chose until the signature says otherwise.

## OAuth and third-party code

A public client (a single-page app, a mobile app) uses the authorization code flow with PKCE. A script loaded from a CDN carries a subresource integrity hash. Why: a public client can't keep a secret, and a CDN can be changed under you.

## WebViews

A mobile WebView starts with JavaScript off, loads only an allowlist of HTTPS hosts, and has no access to local files. Turn on only what the page needs. Why: a WebView with a bridge to the app runs the page's script with the app's power.

## Production config

The app refuses to start in production when debug mode is on or a configured URL that should be remote uses `http://`. Why: a debug page and a plain-text link are the cheapest leaks to ship by accident.

## Secrets

Read secrets from the environment or a secret store. A secret that reached a remote is compromised when it landed: revoke it, issue a new one, then remove it from history. Why: deleting the line leaves the old value in the clone that already fetched it.

CI reaches a cloud account by assuming a short-lived role (OpenID Connect federation) rather than with a stored long-lived key. Why: a stored key works for anyone who reads it, for as long as it exists.

## XML

Parse XML with DTDs and external entities disabled. Why: an entity can read a local file or make the parser fetch a URL.

## Outbound calls

Set a timeout and a maximum response size on every call the server makes. Why: an open connection or an unbounded body is a way to stall the process.

## Log injection

Reject or encode carriage return and line feed in any value written to a log. Why: a newline becomes a second log line that the attacker wrote.

## Session fixation

When authentication succeeds, issue a new session id and discard the one from before login. Why: an id the attacker chose before login would otherwise stay valid after it.
