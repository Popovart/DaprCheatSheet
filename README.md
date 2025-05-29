# 🔧 Dapr + .NET | Command Cheat‑Sheet

> **One‑pager** for day‑to‑day diagnostics, process control and quick restarts—no hard‑coded names, ready for copy‑paste.

---

## 📑 Table of Contents

1. [Quick stop & free a port](#quick-stop--free-a-port)
2. [Port diagnostics](#port-diagnostics)
3. [Process inspection](#process-inspection)
4. [Killing processes](#killing-processes)
5. [Starting a Dapr app](#starting-a-dapr-app)
6. [Status & connectivity](#status--connectivity)
7. [Handy env vars](#handy-env-vars)

---

## Quick stop & free a port

```bash
# Gracefully stop sidecar + app by app‑id
dapr stop --app-id <APP_ID>

# Still occupied? Find the listener and kill it
lsof -iTCP -sTCP:LISTEN -P | \
  grep :<PORT> | \
  awk '{print $2}' | \
  xargs kill          # SIGTERM
```

---

## Port diagnostics

```bash
# Who’s listening on a specific port?
lsof -i :<PORT>

# List every LISTEN socket
lsof -iTCP -sTCP:LISTEN -P
```

---

## Process inspection

```bash
# Filter by keywords
aux | grep -E "(dapr|MyService)" | grep -v grep

# Background jobs of current shell
jobs
```

---

## Killing processes

```bash
# Single PID
kill <PID>

# Mass‑kill by pattern
pkill -f "<PATTERN>"   # e.g. "dapr run"

# Stop ALL Dapr apps
dapr stop
```

---

## Starting a Dapr app

```bash
# YAML‑driven (sidecar + app)
dapr run -f dapr.yaml

# Verbose logs
dapr run -f dapr.yaml --log-level debug

# Run DLL directly (work‑around for `dotnet run`)
dapr run \
  --app-id <APP_ID> \
  --app-port <APP_PORT> \
  --dapr-http-port <HTTP_PORT> \
  --resources-path <PATH_TO_RESOURCES> \
  -- dotnet <PATH_TO_DLL> --urls=http://localhost:<APP_PORT>/
```

---

## Status & connectivity

```bash
# List running Dapr apps
dapr list

# Probe an HTTP endpoint
curl -v http://localhost:<APP_PORT>
```

---

## Handy env vars

| Variable                          | Purpose                                        | Example |
| --------------------------------- | ---------------------------------------------- | ------- |
| `DOTNET_USE_POLLING_FILE_WATCHER` | Fix Hot Reload by switching to polling watcher | `1`     |
| `COMPlus_GCHeapHardLimit`         | Hard memory cap for a .NET process             | `512m`  |

> Replace `<…>` with your own values. Feel free to add more shortcuts as your toolbox grows!
