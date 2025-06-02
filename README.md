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


## Redis Diagnostics for Dapr Pub/Sub

When Redis is used as a Dapr pub/sub store, these commands help you inspect and troubleshoot topic data, consumer groups, and message streams.

1. **Connect to Redis CLI in Docker**

   ```bash
   docker exec -it dapr_redis redis-cli
   ```

   * Adjust `dapr_redis` if your container name differs.

2. **List keys**

   ```bash
   KEYS *              # Show all keys in the DB
   KEYS *pubsub*       # Show keys containing "pubsub"
   KEYS pubsub*        # Show keys starting with "pubsub"
   ```

   * Use `KEYS *` to confirm that your Redis instance is reachable and see every key.
   * Use `KEYS *pubsub*` or `KEYS pubsub*` to filter for Dapr-related streams/metadata.

3. **Check a key’s data type**

   ```bash
   TYPE {key}
   ```

   * Identifies whether `{key}` holds a stream, list, hash, set, etc. For Dapr pub/sub, you’ll typically see `stream`.

4. **Inspect stream contents (topic messages)**

   ```bash
   XRANGE {stream_key} - +            # Show every entry (ID ascending)
   XRANGE {stream_key} {startID} {endID}  # Show entries between specific IDs
   ```

   * Use `-` for the lowest ID and `+` for the highest.
   * Example: `XRANGE pubsuborders - +`

5. **Get stream length and metadata**

   ```bash
   XLEN {stream_key}                   # Number of entries in the stream
   XINFO STREAM {stream_key}           # Info: length, first/last ID, etc.
   XINFO STREAM {stream_key} GROUPS    # List all consumer groups on the stream
   XINFO STREAM {stream_key} CONSUMERS # List consumers in each group
   ```

   * Verifies that messages exist, how many there are, and which group(s) are attached.

6. **Inspect a consumer group’s pending entries**

   ```bash
   XINFO GROUPS {stream_key}                    # Shows each group’s pending count, last-delivered ID
   XINFO CONSUMERS {stream_key} {group_name}    # Shows each consumer’s pending/idle counts
   ```

   * Confirms whether a group (e.g., `dapr-pubsub`) has unacknowledged messages.

7. **Read messages as a specific consumer group**

   ```bash
   XREADGROUP GROUP {group_name} {consumer_name} \
     STREAMS {stream_key} > [COUNT {n}]
   ```

   * `{group_name}`: usually `dapr-pubsub` (default).
   * `{consumer_name}`: any unique identifier (e.g., `consumer-1`).
   * `>` means “only new entries not yet delivered to this group.”
   * Example:

     ```bash
     XREADGROUP GROUP dapr-pubsub consumer-1 STREAMS pubsuborders >
     ```

8. **Acknowledge processed messages**

   ```bash
   XACK {stream_key} {group_name} {id1} [id2 ...]
   ```

   * Marks listed IDs as processed in `{group_name}`. If Dapr fails to acknowledge, messages remain “pending.”

9. **Trim or delete old/pending entries**

   ```bash
   XTRIM {stream_key} MAXLEN {count}    # Keep only the most recent {count} entries
   XTRIM {stream_key} MINID {id}        # Drop entries with ID < {id}
   XDEL {stream_key} {id1} [id2 ...]    # Remove specific message IDs
   ```

   * Use carefully—trimming or deleting pending messages can clear a backlog but may also drop unprocessed events.

10. **Remove a misbehaving consumer group**

    ```bash
    XGROUP DESTROY {stream_key} {group_name}
    ```

    * Deletes `{group_name}` and all pending entries. Dapr will recreate the group on next publish if configured.

11. **Delete a topic stream entirely**

    ```bash
    DEL {stream_key}
    ```

    * Clears all messages and group metadata. Dapr will recreate the stream upon next publish.

---

*Use these commands to verify that messages are published correctly, consumer groups are processing and acknowledging as expected, and to clear any stuck or pending entries. Proceed with caution when trimming or deleting data in production.*

