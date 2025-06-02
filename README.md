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

6. **Stream Commands**

   * **Add an entry to a stream**

     ```bash
     XADD {stream_key} * field1 value1 field2 value2
       # Add a new entry to the stream; '*' lets Redis generate the ID
       # Example: XADD mystream * sensor-id 1234 temperature 19.8
     ```
   * **Read entries from a stream**

     ```bash
     XRANGE {stream_key} - +            # Read all entries in ascending ID order
     XRANGE {stream_key} {start} {end}  # Read entries between specific IDs
       # Use '-' for minimum, '+' for maximum.
       # Example: XRANGE mystream 1609459200000-0 1609462800000-0
     ```
   * **Read entries in reverse order**

     ```bash
     XREVRANGE {stream_key} + -         # Read all entries in descending ID order
     XREVRANGE {stream_key} {end} {start}
       # Swap start/end when using explicit IDs.
       # Example: XREVRANGE mystream 1609462800000-0 1609459200000-0
     ```
   * **Trim a stream to a maximum length**

     ```bash
     XTRIM {stream_key} MAXLEN {count}  # Keep only the most recent {count} entries
       # Example: XTRIM mystream MAXLEN 1000
     XTRIM {stream_key} MINID {id}      # Discard entries with IDs older than {id}
       # Example: XTRIM mystream MINID 1609459200000-0
     ```
   * **Read new entries (blocking or non-blocking)**

     ```bash
     XREAD STREAMS {stream_key} {last_id} [COUNT {count}] [BLOCK {milliseconds}]
       # Read entries with ID > {last_id}; BLOCK waits up to given ms for new entries.
       # Example (non-blocking): XREAD STREAMS mystream 0-0 COUNT 10
       # Example (blocking):    XREAD BLOCK 5000 STREAMS mystream 0-0
     ```
   * **Consumer groups**

     * *Create a consumer group*

       ```bash
       XGROUP CREATE {stream_key} {group_name} $ MKSTREAM
         # Create group {group_name} starting at the latest ID ($).
         # MKSTREAM creates {stream_key} if it does not exist.
         # Example: XGROUP CREATE mystream mygroup $ MKSTREAM
       ```
     * *Read from a consumer group*

       ```bash
       XREADGROUP GROUP {group_name} {consumer_name} \
         STREAMS {stream_key} {last_id} [COUNT {count}] [BLOCK {milliseconds}]
         # {last_id} usually '>' to fetch new entries never delivered to this group.
         # Example: XREADGROUP GROUP mygroup consumer1 STREAMS mystream >
       ```
     * *Acknowledge processed entries*

       ```bash
       XACK {stream_key} {group_name} {id1} [id2 ...]
         # Acknowledge that entries with given IDs have been processed.
         # Example: XACK mystream mygroup 1609459200000-0
       ```
     * *Get consumer group info*

       ```bash
       XINFO GROUPS {stream_key}     # List all groups for the stream
       XINFO CONSUMERS {stream_key} {group_name}  # List consumers in a group
       ```
   * **Delete entries from a stream**

     ```bash
     XDEL {stream_key} {id1} [id2 ...]  # Delete specified entry IDs
       # Example: XDEL mystream 1609459200000-0
     ```
   * **Stream metadata**

     ```bash
     XLEN {stream_key}                     # Get the number of entries in the stream
       # Example: XLEN mystream
     XINFO STREAM {stream_key}             # Get general info: length, first/last ID, etc.
       # Example: XINFO STREAM mystream
     XINFO STREAM {stream_key} CONSUMERS   # Alias for XINFO CONSUMERS in a consumer group context
     XINFO STREAM {stream_key} GROUPS      # Alias for XINFO GROUPS in a consumer group context
     ```

---

*End of Redis Commands section*
