# Security (stdlib)

Read when the code takes user input, touches files or paths, makes tokens, uses crypto, sets cookies or serves HTTP. Ask before changing auth, crypto or security behaviour. Checked on go1.27.1 unless marked.

## Files and paths
- **`filepath.Join` plus `strings.HasPrefix` is not a sandbox.** `filepath.Join("/var/www", "../www-evil/x")` is `/var/www-evil/x`, and it passes `HasPrefix("/var/www")`. Open user-named files through `os.Root`:
  ```go
  root, err := os.OpenRoot("/var/www")
  if err != nil { return err }
  defer root.Close()
  f, err := root.Open(userPath) // fails on "..", absolute paths and escaping symlinks
  ```
  `os.Root` does not block bind mounts, `/proc` files or device files. `filepath.IsLocal` is a lexical check only.
- Cap decompressed size with `io.LimitReader` at limit+1 and return an error when the limit is hit. Why: `io.Copy` treats EOF as success, so a silent cap truncates data instead of rejecting the bomb.
- Pass user values to `exec.Command` as separate arguments, never through `sh -c`, and allowlist what they may be.

## Randomness and secrets
- Tokens and keys come from `crypto/rand` (`rand.Text()` for a random string, `rand.Read`). Seeding `math/rand` from `crypto/rand` still gives a predictable stream.
- Compare MACs with `hmac.Equal`. `subtle.ConstantTimeCompare` returns 0 at once when the lengths differ, so check the length is the expected one first.
- AES-GCM: a random 96-bit nonce is safe only up to about 2^32 messages per key. For high volume use a counter nonce or rotate keys. Never reuse a nonce with the same key. (Bound from the GCM spec, not run here.)
- Passwords: bcrypt (`GenerateFromPassword` returns `ErrPasswordTooLong` above 72 bytes, x/crypto v0.57.0), scrypt or argon2id via `golang.org/x/crypto`. Take the parameters from the current OWASP cheat sheet, not from memory.
- JWT: fix the accepted algorithm on the server and reject any other, including `none`. Never trust the token's own `alg` header.

## HTTP
- HTML goes through `html/template`. `text/template` does no escaping.
- Cookie-authenticated browser endpoints: wrap the mux with `http.NewCrossOriginProtection()` for CSRF (it checks `Sec-Fetch-Site`, then Origin against Host).
- Session cookies: `HttpOnly`, `Secure`, `SameSite=Lax` or `Strict`, and an empty `Domain` so the cookie stays host-only. Setting `Domain` shares it with every subdomain.
- Cap request bodies with `http.MaxBytesReader` before decoding.
- Set `ReadHeaderTimeout` on every `http.Server`. A zero-value server has no timeouts at all.
- Errors to clients: a fixed body and status. Driver errors (`*pgconn.PgError`), stack traces and `err.Error()` stay in the log.
- pprof and other debug handlers go on a separate mux bound to localhost, never on the public one. Why: importing `net/http/pprof` registers on `http.DefaultServeMux`.

## Logs
- Never log secrets, tokens, passwords, full request bodies or unredacted personal data. Allowlist the attributes you log.
- Implement `slog.LogValuer` on types that hold secrets so they log as `"[redacted]"` even when someone logs the whole struct.

## SQL
Values always go through placeholders (`$1` or `?` depending on the driver), and identifiers through an allowlist. Everything else is in the database skill.
