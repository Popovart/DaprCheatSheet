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


## Redis Commands

Use these commands to connect to a Redis instance and manage keys directly from the CLI.

1. **Open Redis CLI inside a Docker container**

   ```bash
   docker exec -it dapr_redis redis-cli
   ```

   * Replace `dapr_redis` with the actual container name if different.

2. **List keys**

   ```bash
   KEYS *              # Show all keys
   KEYS *pubsub*       # Search for keys containing "pubsub"
   KEYS pubsub*        # Search for keys starting with "pubsub"
   ```

3. **Inspect a key’s type and value**

   ```bash
   TYPE {key}          # Get the data type of the key
   GET {key}           # Retrieve string value (for String type)
   LRANGE {key} 0 -1   # Retrieve list elements from start to end (for List type)
   HGETALL {key}       # Retrieve all field-value pairs (for Hash type)
   SMEMBERS {key}      # Retrieve all members of a set (for Set type)
   ```

   * Replace `{key}` with the actual key name you want to inspect.

4. **Delete keys**

   ```bash
   DEL {key}           # Remove a specific key
   ```

5. **Flush entire database**

   ```bash
   FLUSHALL            # Remove all keys in the current Redis database (use with caution)
   ```

