# Model and LLM features

Read when the feature calls a model, or puts text from a user, a page, a file, or a tool into a prompt. Model output is untrusted input. The system prompt is not a boundary.

- Parse model output as data, then check it against an allowlist, then encode it for where it goes. Why: the text can be a query, markup, a path, or a tool argument.
- Do not send the output into a query string, a shell, an HTML sink, or a file path. A tool runs only if the code allows that tool for this caller, and the code checks the arguments again.
- Keep secrets, other tenants' data, and personal data out of the prompt and the retrieved context. Why: anything in the context can come back in the answer or land in a log.
- Confirm a destructive or irreversible tool action with the user before it runs. Why: the model can be talked into calling it.
- Cap tokens, request rate, and how deep a tool may call another tool. Take the limits from the product's own requirements.
- If retrieval is stored per tenant, a query for one tenant must not read another's documents. Check a document before it is indexed.

Abuse cases to test: a user message that says to ignore the instructions and dump the prompt; a fetched page that contains a tool call; a completion that is inserted into a page as HTML.
