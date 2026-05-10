Run `npx @marp-team/marp-cli --server --port $PORT slides/` from the workspace root. Use `$PORT` from the environment — do not invent a port number.

The server is long-running -- leave it running in the background (the harness will auto-background it). Do not ask when to stop it.

If a different markdown server (e.g. md-serve) is already bound to `$PORT`, stop it first.
