Run the Marp slide server for `slides/` from the workspace root on `$PORT`. Use `$PORT` from the environment -- do not invent a port number. Do not ask when to stop it.

1. If anything already listens on `$PORT` (e.g. md-serve), stop it first using the "To stop" steps below.
2. Start it in the background, detached in its own process group, so this command returns immediately (never run it in the foreground):
   ```
   setsid nohup npx @marp-team/marp-cli --server --port "$PORT" slides/ > "/tmp/marp-$PORT.log" 2>&1 < /dev/null &
   ```
3. Confirm it is up: retry for up to 60 seconds (npx may download first) until this prints `200`:
   ```
   curl -s -o /dev/null -w '%{http_code}' "http://localhost:$PORT/"
   ```
   If it never does, show the tail of `/tmp/marp-$PORT.log` instead of claiming success.

To stop it: `npx` starts several processes (npm, sh, node), so killing one PID leaves the server running. Kill the whole process group of whatever listens on `$PORT`:
```
pid=$(lsof -t -iTCP:"$PORT" -sTCP:LISTEN)
kill -- -"$(ps -o pgid= -p "$pid" | tr -d ' ')"
```
Then run the same `curl` until it prints `000`. Only report "stopped" after that check passes.
