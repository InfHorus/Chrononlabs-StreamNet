<div align="center">

<h1>⚡ ChrononLabs-StreamNet</h1>
<h3>Fast, Reliable & Secure Garry's Mod Networking</h3>

<p><em>A lightweight, optimized, single-file streaming networking library for Garry's Mod.</em></p>

<img alt="Garry's Mod" src="https://img.shields.io/badge/Garry's%20Mod-Lua-blue">
<img alt="Single File" src="https://img.shields.io/badge/single--file-%E2%9C%93-success">
<img alt="Up to 3x Faster" src="https://img.shields.io/badge/up%20to-3x%20faster-orange">
<img alt="Secure by Policy" src="https://img.shields.io/badge/secure-by%20policy-blueviolet">

</div>

---

ChrononLabs-StreamNet makes networking cleaner, safer, and more reliable across all kinds of projects. Use it for simple addon messages, structured data sync, raw binary transfers, or very large payloads that would normally be painful with the default `net` library. It also works as a better-organized, more optimized layer for everyday communication.

## Benchmark

vNet vs NetStream vs ChrononLabs-StreamNet, client to server upload, all libraries at stock defaults.

<img width="741" height="190" alt="image" src="https://github.com/user-attachments/assets/89d1a9a0-c2f6-419c-b2b2-0dd7f0158886" />

## Why it exists

Garry's Mod `net` is fine for small one-off messages, but it gets painful when an addon needs larger data, safer client messages, progress tracking, retries, or cleaner control over how data is sent. The normal system has a practical limit around 64 KB per message.

ChrononLabs-StreamNet streams data in chunks (default payload limit 8 MB, raisable for trusted use cases), paces the transfer, validates the data, retries missing chunks, and rebuilds the full payload before your callback runs. On top of that you get receive policies, priority, compression, progress callbacks, cancellation, timeout handling, stats, and one API for both client→server and server→client messages.

Those same features also make your server much harder to abuse. Because every incoming message is gated by a receive policy you control, spammers, exploiters, and oversized or wrong-way payloads get rejected before your handler ever runs. Each policy closes off a common attack or mistake:

- **`Direction`** (`any`, `client_to_server`, `server_to_client`): reject messages coming from the wrong side, so clients can't trigger server-only handlers.
- **`MaxBytes`**: cap the original payload size (before compression) to stop oversized and memory-bombing transfers.
- **`MaxInFlight`**: limit how many active incoming transfers a single peer can have per message, stopping parallel flooding.
- **`Cooldown`**: enforce a minimum delay between starts of the same message from one peer, throttling rapid spam.
- **`MaxPerWindow`**: limit accepted transfer starts per peer over a sliding time window, catching slower sustained spam that slips past a cooldown.
- **`RequireReady`**: ignore messages until a joining client has sent the internal ready signal, blocking early/forged sends during load.
- **`RequireUsergroup`**: require `admin`, `superadmin`, or a custom usergroup before a server handler runs, so privileged messages can't be faked by normal players.

On top of policies, checksums reject corrupted chunks, duplicate protection ignores replayed retry packets, and timeouts clean up stalled transfers. Together these turn the transport into a strong first line of defense against spam, oversized payloads, corrupted data, repeated sends, and exploiters flooding or attacking your receivers.

The policies above are only the security layer. For the full list of everything the library does, see [Main advantages](#main-advantages).

This does **not** replace an anticheat, permission checks, or server-side validation. You still need to check what a client is allowed to do. It just gives you a safer transport to build on, so your addon code can focus on the data instead of re-implementing chunking and retries in every project.

Useful for inventory data, save data, debug dumps, admin tools, UI state, generated cache data, raw binary data, anticheat reports, AI telemetry, file-like transfers, and any addon where normal net messages start to feel too limited or messy.

## Main advantages

- Single-file library that can live in one shared autorun file.
- Unified API with `Receive`, `Send`, `SendEx`, `SendRaw`, `Broadcast`, `BroadcastEx`, `Request`, `Respond`, `Replicate`, `OnReplicated`, and `GetReplicated`.
- Two-way networking for client to server and server to client messages.
- Works for small addon messages, large payloads, and file-like transfers.
- Cleaner structure than manually managing `net.Start`, `net.Write*`, and `net.Receive`.
- Automatic payload chunking for messages that exceed normal net comfort limits.
- Per-peer pacing with `BytesPerSecond`, `BurstBytes`, `Window`, and `MaximumPacketsPerThink`.
- Deferred completion handling with `MaximumCompletionsPerThink`, so many finished transfers do not all decode and call handlers in the same net receive frame.
- Congestion control for unreliable transfers, so aggressive sends can back off instead of flooding weak links.
- Up to around 3x faster than other large-payload networking libraries such as NetStream and vNet in local tests.
- Transfer priority with `Priority = "high"`, `Priority = "normal"`, and `Priority = "low"`.
- Priority aging with `PriorityAgingInterval` so lower priority transfers still get chances to send.
- Optional compression with `util.Compress` and `CompressAt`.
- Raw binary streaming with `SendRaw` when you already have your own encoded format.
- Structured argument serialization for nil, booleans, numbers, strings, tables, Vector, Angle, Color, and Entity values.
- Pure-Lua IEEE-754 binary64 double encoding for floating point and non-Int32 Lua numbers, avoiding file-backed `File:WriteDouble` and `File:ReadDouble` serialization workarounds.
- Serializer safety limits with `MaximumTablePairs` and `MaximumTableDepth`.
- Payload size protection with `MaximumPayloadBytes` and safe chunk sizing.
- ACK/NACK recovery, where received chunks are confirmed, and missing or corrupted chunks are requested again.
- Automatic retry with `RetryInterval` and `MaximumRetries`.
- Transfer timeout handling with `Timeout`.
- Full payload validation before delivery.
- Chunk and complete-payload checksum validation.
- Duplicate delivery protection for late retry packets.
- Finished-transfer memory with `FinishedIncomingTtl` to ignore late duplicates safely.
- Completion callbacks with `OnComplete`.
- Progress callbacks for outgoing transfers with `OnProgress` and `ProgressInterval`.
- Request/response helpers with correlation, timeout, duplicate-reply protection, and fast failure.
- Replicated values with `Replicate`, `OnReplicated`, `GetReplicated`, and late-join sync for large config/state tables.
- Outgoing transfer lookup with `GetTransfer` and `GetTransfers`.
- Outgoing transfer cancellation with `Cancel` and `CancelAll`.
- Runtime stats with `GetStats`, `ResetMetrics`, and `chrononlabs_streamnet_stats`.
- Optional client-ready queueing with `QueueUntilClientReady`.
- Helpful error messages that include short fixes when something is misconfigured.
- Reusable profiles for receive policies and send options.
- Optional receive policies for per-message safety limits.
- Receive `Direction` policies: `any`, `client_to_server`, or `server_to_client`.
- Receive `MaxBytes` policies to cap the original payload size before compression.
- Receive `MaxInFlight` policies to limit active incoming transfers per peer and message.
- Receive `Cooldown` policies to limit how often one peer can start the same message.
- Receive `RequireReady` policies to wait until a joining client has sent the internal ready signal.
- Receive `RequireUsergroup` policies to require admin, superadmin, or a custom usergroup before server handlers run.
- Receive `MaxPerWindow` policies to limit accepted transfer starts per peer and message over a sliding time window.
- Useful for basic addons, advanced systems, admin tools, anticheat systems, AI telemetry, save systems, and file-like transfers.

## Installation

Place the file somewhere shared, for example:

```lua
lua/autorun/chrononlabs-stream-net.lua
```

The file handles client distribution automatically when loaded server-side. No `util.AddNetworkString` calls are needed. Message names are managed internally.

## Delivery model

ChrononLabs-StreamNet provides **guaranteed complete delivery**: a message reaches your callback only after the full payload has been received, validated, assembled, and decompressed if needed. Missing chunks are requested again; if a transfer cannot recover, it **fails** instead of silently delivering incomplete or corrupted data. This makes it suitable for systems where partial delivery is not acceptable.

> **CRC note:** checksums catch accidental corruption / payload mismatch, not malicious tampering. Always keep validating client data server-side.

## Quick start

**Server**

```lua
ChrononLabsStreamNet.Receive ("ClientHello", function (ply, message)
    print ("Client said:", ply, message)
    ChrononLabsStreamNet.Send ("ServerReply", ply, "Hello from the server")
end)
```

**Client**

```lua
ChrononLabsStreamNet.Receive ("ServerReply", function (message)
    print ("Server replied:", message)
end)

ChrononLabsStreamNet.Send ("ClientHello", "Hello from the client")
```

Receive callbacks get `(ply, ...)` on the server and `(...)` on the client. `Send` on the server takes the target player after the name; on the client it does not.

## The API at a glance

| Call | Use it when |
| --- | --- |
| `Send (name, [target], ...)` | Small structured message, no options needed. |
| `SendEx (name, [target], options, ...)` | A structured send that needs priority, compression, progress, timeout, or custom pacing. |
| `SendRaw (name, [target], bytes, options)` | You already have bytes/JSON/compressed data/file data/your own format. |
| `Broadcast (name, ...)` | Server sends the same structured message to every player. |
| `BroadcastEx (name, options, ...)` | A broadcast that needs options, a profile, or `OnAllComplete`. |
| `Request (name, [target], data, options, cb)` | One side asks one peer for data and expects exactly one answer. |
| `Respond (name, policy, cb)` | The other end of a `Request`. |
| `Replicate / OnReplicated / GetReplicated` | Server owns a value current and future clients should cache. |
| `Receive (name, [policy], cb)` | Register a handler, optionally with safety limits. |
| `GetTransfer / GetTransfers` | Inspect running outgoing transfers (for UI/debug). |
| `Cancel / CancelAll` | Stop running outgoing transfers. |
| `GetStats / ResetMetrics` | Read or reset runtime metrics. |
| `DefineProfile` | Reuse a policy/options table by name. |
| `SetConfig` | Change global defaults. |

`options` and `policy` arguments can be either an inline table or the name of a profile (see [Profiles](#profiles)). `Receive` is also exposed as `On` and `Register`.

## Sending data

### Structured values: `Send` / `SendEx`

`Send` serializes nil, booleans, numbers, strings, tables, `Vector`, `Angle`, `Color`, and `Entity` values automatically.

```lua
-- Server → one client
ChrononLabsStreamNet.Send ("PlayerData", ply, {
    Name = "Player",
    Level = 25,
    Position = Vector (100, 200, 300),
    Stats = { Health = 100, Armor = 50 }
})
```

`SendEx` is the same but takes an options table (or profile name) before the payload:

```lua
ChrononLabsStreamNet.SendEx ("MenuState", ply, {
    Priority = "high",
    Compress = true
}, menuState)
```

> Functions cannot be serialized. To send tables that may contain functions (e.g. `hook.GetTable ()`), sanitize them first, see [the worked example](#example-2-large-sanitized-debug-dump).

### Raw bytes: `SendRaw`

Use raw mode when you already have an encoded format, a compressed blob, generated file content, or any binary payload.

```lua
-- Client → server
local data = string.rep ("A", 300000)

ChrononLabsStreamNet.SendRaw ("UploadBlob", data, {
    ChunkSize = 16384,
    BytesPerSecond = 128 * 1024,
    Window = 8,
    OnComplete = function (ok, reason)
        print ("Upload complete:", ok, reason)
    end
})
```

```lua
-- Server
ChrononLabsStreamNet.Receive ("UploadBlob", function (ply, bytes)
    print ("Received upload from", ply, "size:", #bytes)
end)
```

### Broadcasting (server only)

```lua
ChrononLabsStreamNet.Broadcast ("GlobalAnnouncement", {
    Title = "Server Update",
    Message = "A new system has been loaded."
})

ChrononLabsStreamNet.BroadcastEx ("GlobalConfig", {
    OnAllComplete = function (summary)
        print ("Broadcast done:", summary.Completed, "failed:", summary.Failed)
    end
}, configTable)
```

### Multiple targets and `OnAllComplete`

`SendEx` and `SendRaw` accept a player table as the target. `OnAllComplete` fires once after every resolved target finishes:

```lua
ChrononLabsStreamNet.SendEx ("BigConfig", players, {
    OnAllComplete = function (summary)
        print ("Done:", summary.Completed, "failed:", summary.Failed)
        PrintTable (summary.Results)
    end
}, configTable)
```

## Receiving safely (receive policies)

`Receive` accepts an optional policy table before the callback, for messages that need stricter limits than the global defaults. All fields are optional.

```lua
ChrononLabsStreamNet.Receive ("AvatarUpload", {
    Direction = "client_to_server",
    MaxBytes = 256 * 1024,
    MaxInFlight = 1,
    Cooldown = 5,
    RequireReady = true,
    RequireUsergroup = "admin",
    MaxPerWindow = { Limit = 3, Window = 60 }
}, function (ply, bytes)
    print ("Avatar upload from", ply, "bytes:", #bytes)
end)
```

| Field | Effect |
| --- | --- |
| `Direction` | `any`, `client_to_server`, or `server_to_client`. |
| `MaxBytes` | Caps the original payload size **before** compression. |
| `MaxInFlight` | Limits active incoming transfers for that message, per peer. |
| `Cooldown` | Minimum seconds between accepted starts of that message, per peer. |
| `RequireReady` | Server waits until the joining client has sent the internal ready signal. |
| `RequireUsergroup` | Server-side gate for client→server messages. `admin` uses `IsAdmin()`, `superadmin` uses `IsSuperAdmin()`, custom groups compare case-insensitively against `GetUserGroup()`. |
| `MaxPerWindow` | `{ Limit, Window }`, caps accepted starts per peer/message over a sliding time window. |

The transport can prove a payload arrived intact; it cannot decide whether the player was allowed to send it. Always keep your own permission and sanity checks.

## Profiles

Profiles let you define a policy/options table once and reuse it by name, instead of repeating the same table everywhere.

```lua
ChrononLabsStreamNet.DefineProfile ("SmallClientAction", {
    Direction = "client_to_server",
    MaxBytes = 4096,
    MaxInFlight = 1,
    Cooldown = 0.25,
    RequireReady = true,
    Priority = "high"
})

ChrononLabsStreamNet.Receive ("UseItem", "SmallClientAction", function (ply, itemId)
    print ("Use item:", ply, itemId)
end)

-- Client
ChrononLabsStreamNet.SendEx ("UseItem", "SmallClientAction", itemId)
```

`Receive`, `SendEx`, `SendRaw`, `BroadcastEx`, `Request`, and `Respond` accept a profile name anywhere they take a policy/options table. `Send` and `Broadcast` don't take profiles because they have no options slot.

## Request / Response

Use `Request`/`Respond` when one side asks for data and expects exactly one answer. The library creates the internal request/reply messages, correlates the reply, enforces the timeout, and ignores duplicate or late replies.

`Request` is point-to-point. On the server, pass exactly one valid player target. The request payload is one value; replies may return multiple values via `reply (true, ...)` or fail logically with `reply (false, reason, ...)`. `Timeout` in the options is the round-trip timeout (not the transport timeout). `OnComplete` is reserved internally, but `OnProgress` is allowed if the request upload is large.

**Client asks the server**

```lua
ChrononLabsStreamNet.Request ("GetInventory", { IncludeEquipment = true }, {
    Timeout = 10
}, function (ok, responseOrReason)
    if not ok then print ("Inventory request failed:", responseOrReason) return end
    PrintTable (responseOrReason)
end)
```

```lua
ChrononLabsStreamNet.Respond ("GetInventory", {
    Direction = "client_to_server",
    Cooldown = 1,
    MaxBytes = 4096
}, function (ply, request, reply)
    if not IsValid (ply) then return reply (false, "invalid player") end
    if not CanOpenInventory (ply) then return reply (false, "not allowed") end

    reply (true, {
        Items = BuildInventory (ply),
        Equipment = request.IncludeEquipment and BuildEquipment (ply) or nil
    })
end)
```

**Server asks one client**, same shape, but the target player comes first and the `Respond` callback has no `ply`:

```lua
ChrononLabsStreamNet.Request ("GetClientState", ply, { IncludeHud = true }, {
    Timeout = 5
}, function (ok, responseOrReason)
    if not ok then print ("Client state request failed:", responseOrReason) return end
    PrintTable (responseOrReason)
end)
```

```lua
ChrononLabsStreamNet.Respond ("GetClientState", {
    Direction = "server_to_client",
    MaxBytes = 16 * 1024
}, function (request, reply)
    reply (true, BuildClientState (request))
end)
```

## Replicated values

Server-owned values cached on every client. Useful for config, shop data, rules, HUD state, or large tables that late joiners must receive.

```lua
-- Server
ChrononLabsStreamNet.Replicate ("ServerConfig", configTable)
ChrononLabsStreamNet.ClearReplicated ("ServerConfig")   -- or Replicate("ServerConfig", nil)

-- Client
ChrononLabsStreamNet.OnReplicated ("ServerConfig", function (value, name, cleared)
    print ("Config changed:", name, cleared)
end)

local config = ChrononLabsStreamNet.GetReplicated ("ServerConfig", {})
```

`OnReplicated` fires on initial delivery and on later updates. If a replication transfer ultimately fails, the client may stay stale until the server calls `Replicate` again.

## Priority and pacing

Priority decides which outgoing transfers get pumped first when a peer has multiple active sends. Use `high` for data the player is waiting on right now (menu state), `low` for work that can wait (debug dumps, cache).

```lua
ChrononLabsStreamNet.SendEx ("MenuState", ply, { Priority = "high" }, state)
ChrononLabsStreamNet.SendRaw ("DebugDump", ply, payload, { Priority = "low" })
```

Don't mark everything high. If everything is high priority, nothing is. `high` still works for large payloads, but it will delay other messages while it sends.

`PriorityAgingInterval` (default `2`) controls how quickly waiting transfers earn a boost, so low-priority transfers don't sit forever behind higher-priority ones.

Pacing is **per player** and set via `SpeedProfile` (or individual knobs). Built-in profiles:

```lua
conservative = { BytesPerSecond = 96 * 1024,       BurstBytes = 64 * 1024,       Window = 6 }
balanced     = { BytesPerSecond = 1 * 1024 * 1024, BurstBytes = 512 * 1024,      Window = 24 }  -- default
fast         = { BytesPerSecond = 2 * 1024 * 1024, BurstBytes = 512 * 1024,      Window = 32 }
lightning    = { BytesPerSecond = 3 * 1024 * 1024, BurstBytes = 1 * 1024 * 1024, Window = 48 }
```

```lua
ChrononLabsStreamNet.SetConfig ("SpeedProfile", "fast")
ChrononLabsStreamNet.SetConfig ("Window", 24)   -- override one knob after the profile
```

In local testing (4 MiB unreliable transfer, 16 KiB chunks) these measured roughly `0.5`, `1`, `2`, and `3` MiB/s respectively. `lightning` is not recommended for heavily populated servers (60+ players).

## Transfer control and progress

Large outgoing transfers can be inspected or cancelled while running.

```lua
local id = ChrononLabsStreamNet.SendRaw ("LargeDownload", ply, data, {
    ProgressInterval = 0.25,
    OnProgress = function (transfer)
        print ("Progress:", transfer.AckCount .. "/" .. transfer.TotalChunks)
    end,
    OnComplete = function (ok, reason, transfer)
        print ("Finished:", ok, reason)
    end
})

local transfer  = ChrononLabsStreamNet.GetTransfer (id, ply)   -- one transfer
local transfers = ChrononLabsStreamNet.GetTransfers (ply)      -- all for that peer

ChrononLabsStreamNet.Cancel (id, ply, "(MyAddon): Download cancelled by user.")
local cancelled = ChrononLabsStreamNet.CancelAll (ply, "(MyAddon): All downloads cancelled.")
```

`CancelAll` returns the number of transfers it stopped. On the **client**, drop the player argument:

```lua
ChrononLabsStreamNet.Cancel (id, "(MyAddon): Upload cancelled by user.")
ChrononLabsStreamNet.CancelAll ("(MyAddon): All uploads to the server cancelled.")
```

If a transfer could start, `SendRaw`/`SendEx` return a transfer id; a falsy return means it could not start.

## Stats

```txt
chrononlabs_streamnet_stats
```

```lua
PrintTable (ChrononLabsStreamNet.GetStats ())   -- great for live UI updates
ChrononLabsStreamNet.ResetMetrics ()            -- zeroes lifetime counters, leaves active transfers alone
```

Stats include active outgoing transfers, unacknowledged chunks, remaining outgoing bytes, active incoming transfers, and lifetime metric snapshots.

## Configuration reference

### Per-transfer options (`SendEx` / `SendRaw`)

```lua
local options = {
    ChunkSize        = 16384,   -- max data bytes per chunk; keep safely below the net message limit
    Compress         = true,    -- enable util.Compress for this transfer
    CompressAt       = 8192,    -- only compress when the original payload is at least this many bytes
    BytesPerSecond   = 1 * 1024 * 1024,  -- per-transfer pacing override
    BurstBytes       = 512 * 1024,       -- per-transfer burst override
    Window           = 12,      -- max unacknowledged chunks in flight
    RetryInterval    = 0.75,    -- wait before retrying an unacknowledged chunk
    Timeout          = 25,      -- fail after this many seconds without progress
    MaximumRetries   = 20,      -- max retries per chunk
    ReliableData     = false,   -- false = unreliable + ACK/NACK repair (recommended for large data)
                                -- true  = reliable chunks (can block other reliable messages on big transfers)
    Priority         = "normal",-- "high", "normal", or "low"
    ProgressInterval = 0.25,    -- min seconds between OnProgress calls
    OnProgress       = function (transfer) end,            -- fires as acked chunks change
    OnComplete       = function (ok, reason, transfer) end,-- fires on completion or failure
    OnAllComplete    = function (summary) end              -- multi-target sends only
}
```

> Under load, large unreliable chunks can cause extra resends, lower `ChunkSize` first if drops are frequent. `ReliableData = true` can help small or latency-sensitive transfers, but large reliable transfers may block other reliable net messages. Treat it as a trade-off, not a default.

### Useful `transfer` fields (inside `OnComplete` / `OnProgress`)

```lua
transfer.Id, transfer.Name, transfer.Mode, transfer.Peer
transfer.RawSize, transfer.PackedSize, transfer.Compressed
transfer.Checksum            -- numeric CRC32
transfer.ChunkSize, transfer.TotalChunks, transfer.AckCount, transfer.InFlightCount
transfer.RetryInterval, transfer.Timeout, transfer.MaximumRetries, transfer.Window, transfer.ReliableData
transfer.CreatedAt, transfer.LastProgress, transfer.ProgressInterval
transfer.LastProgressCallback, transfer.LastProgressAckCount, transfer.Done

-- Detailed internal tables (rarely needed): Sent, Retries, Acked, InFlight, NackQueue, NackSeen
-- Avoid printing transfer.Data for huge payloads.
```

### Global defaults (`SetConfig`)

Set these once before heavy use.

```lua
ChrononLabsStreamNet.SetConfig ("MaximumNetMessageBytes", 60000)
ChrononLabsStreamNet.SetConfig ("ChunkSize", 16384)
ChrononLabsStreamNet.SetConfig ("SpeedProfile", "balanced")   -- sets BytesPerSecond/BurstBytes/Window together
ChrononLabsStreamNet.SetConfig ("BytesPerSecond", 1 * 1024 * 1024)
ChrononLabsStreamNet.SetConfig ("BurstBytes", 512 * 1024)
ChrononLabsStreamNet.SetConfig ("Window", 24)
ChrononLabsStreamNet.SetConfig ("RetryInterval", 0.75)
ChrononLabsStreamNet.SetConfig ("Timeout", 20)
ChrononLabsStreamNet.SetConfig ("MaximumRetries", 16)
ChrononLabsStreamNet.SetConfig ("Compress", true)
ChrononLabsStreamNet.SetConfig ("CompressAt", 8192)
ChrononLabsStreamNet.SetConfig ("MaximumPayloadBytes", 8 * 1024 * 1024)
ChrononLabsStreamNet.SetConfig ("MaximumIncomingTransfersPerPeer", 24)
ChrononLabsStreamNet.SetConfig ("MaximumIncomingBytesPerPeer", 32 * 1024 * 1024)
ChrononLabsStreamNet.SetConfig ("MaximumTablePairs", 4096)
ChrononLabsStreamNet.SetConfig ("MaximumTableDepth", 32)
ChrononLabsStreamNet.SetConfig ("AckInterval", 0.035)
ChrononLabsStreamNet.SetConfig ("NackInterval", 0.35)
ChrononLabsStreamNet.SetConfig ("AckBatch", 64)
ChrononLabsStreamNet.SetConfig ("NackBatch", 64)
ChrononLabsStreamNet.SetConfig ("MaximumPacketsPerThink", 24)
ChrononLabsStreamNet.SetConfig ("MaximumCompletionsPerThink", 16)
ChrononLabsStreamNet.SetConfig ("FinishedIncomingTtl", 30)
ChrononLabsStreamNet.SetConfig ("MaximumFinishedIncomingPerPeer", 256)
ChrononLabsStreamNet.SetConfig ("FinishedControlResendInterval", 0.25)
ChrononLabsStreamNet.SetConfig ("PriorityAgingInterval", 2)
ChrononLabsStreamNet.SetConfig ("QueueUntilClientReady", false)
ChrononLabsStreamNet.SetConfig ("RequestTimeout", 15)
ChrononLabsStreamNet.SetConfig ("ResponseMaxBytes", nil)
ChrononLabsStreamNet.SetConfig ("Debug", false)
```

## How delivery works (ACK / NACK / retry / timeout)

You never call ACK or NACK manually. The flow is automatic:

1. Sender splits the payload into chunks.
2. Sender sends chunks according to pacing and window limits.
3. Receiver ACKs chunks that arrived correctly.
4. Receiver NACKs chunks that are missing or corrupted.
5. Sender retries missing chunks.
6. Receiver assembles the full payload only when every chunk is present.
7. Receiver validates the final payload checksum.
8. Receiver calls your `Receive` callback.
9. Receiver sends final completion confirmation.
10. Sender calls `OnComplete`.

Finished incoming transfers are remembered for a short time (`FinishedIncomingTtl`), so the receiver can ignore late duplicate chunks without calling your callback again, while still answering the sender with the right completion/cancel message. If a transfer can't finish before its timeout or retry limit, it fails in a predictable, handleable way.

## Security and best practices

**Do not trust client data just because the transport succeeded.** ChrononLabs-StreamNet improves delivery, pacing, recovery, and structure. It does not replace server-side validation, permission checks, sanity checks, or anticheat logic.

For client→server messages, always validate both that the player is allowed to send the data **and** the contents of the payload itself.

For tiny one-off messages the default `net` library is still fine. Reach for ChrononLabs-StreamNet when you want cleaner structure, better reliability, larger transfers, recovery logic, or a unified API across your project.

## Example use cases

- Addon configuration sync
- Inventory systems
- Character data
- Duplication and build data
- Save and load systems
- Large anticheat reports
- AI or machine learning telemetry
- Custom file transfer
- Admin tools
- Complex UI state syncing
- Server to client cache replication
- Large generated datasets
- Any system where normal net messages become too limited or messy

## Worked examples

> Aimed at larger projects where fine control matters. The library can serialize normal Lua values, tables, numbers, strings, booleans, vectors, angles, colors, and entities, **but not functions**. Tables like `hook.GetTable ()` contain functions, so sanitize them first (Example 2).

### Example 1: Settings sync with completion callback

Sends a structured table from server to one client. Chunking, compression, ACK/NACK, retry, timeout, and final delivery are automatic.

```lua
-- Server
hook.Add ("PlayerInitialSpawn", "ChrononLabsStreamNetExampleSettings", function (ply)
    timer.Simple (3, function ()
        if not IsValid (ply) then return end

        local settings = {
            Version = 1,
            ServerName = GetHostName (),
            Features = { Inventory = true, Anticheat = true, BuildMode = false },
            Limits   = { MaxProps = 500, MaxVehicles = 10, MaxUploads = 3 }
        }

        ChrononLabsStreamNet.SendEx ("ExampleSettingsSync", ply, {
            ChunkSize = 16384,
            Compress = true,
            CompressAt = 8192,
            RetryInterval = 0.75,
            Timeout = 20,
            MaximumRetries = 16,
            Window = 12,
            ReliableData = false,
            OnComplete = function (ok, reason, transfer)
                print ("[SettingsSync] complete:", ok, reason)
                print ("[SettingsSync] chunks acked:", transfer.AckCount .. "/" .. transfer.TotalChunks)
                print ("[SettingsSync] compressed:", transfer.Compressed)
            end
        }, settings)
    end)
end)
```

```lua
-- Client
ChrononLabsStreamNet.Receive ("ExampleSettingsSync", function (settings)
    print ("Received server settings")
    PrintTable (settings)
end)
```

### Example 2: Large sanitized debug dump

Sends big Garry's Mod tables (`concommand.GetTable ()`, `hook.GetTable ()`) without crashing on function values, by converting unsupported values into safe strings first. Useful for debug/admin tools, anticheat telemetry, and developer panels.

**Shared sanitizer** (put it somewhere shared):

```lua
local function ChrononLabsStreamNetSafeCopy (value, depth, seen)
    depth = depth or 0
    seen = seen or {}

    local valueType = type (value)
    if valueType == "nil" or valueType == "boolean" or valueType == "number" or valueType == "string" then
        return value
    end

    if IsEntity and IsEntity (value) then return IsValid (value) and tostring (value) or "NULL Entity" end
    if isvector and isvector (value) then return { Type = "Vector", X = value.x, Y = value.y, Z = value.z } end
    if isangle  and isangle  (value) then return { Type = "Angle",  P = value.p, Y = value.y, R = value.r } end
    if IsColor  and IsColor  (value) then return { Type = "Color",  R = value.r, G = value.g, B = value.b, A = value.a } end

    if valueType == "function" then return "[function]" end
    if valueType ~= "table"    then return "[" .. valueType .. "] " .. tostring (value) end

    if depth >= 8        then return "[max depth]" end
    if seen [value]      then return "[cycle]" end
    seen [value] = true

    local output, count, maximumEntries = {}, 0, 2500
    for key, pairValue in pairs (value) do
        count = count + 1
        if count > maximumEntries then
            output ["__truncated"] = true
            break
        end

        local safeKey = ChrononLabsStreamNetSafeCopy (key, depth + 1, seen)
        if type (safeKey) ~= "string" and type (safeKey) ~= "number" then safeKey = tostring (safeKey) end
        output [safeKey] = ChrononLabsStreamNetSafeCopy (pairValue, depth + 1, seen)
    end

    seen [value] = nil
    return output
end
```

**Client requests, server builds and sends, client receives:**

```lua
-- Client
concommand.Add ("clsnet_request_debug_dump", function ()
    ChrononLabsStreamNet.Send ("ExampleRequestDebugDump", { IncludeConCommands = true, IncludeHooks = true })
end)
```

```lua
-- Server
ChrononLabsStreamNet.Receive ("ExampleRequestDebugDump", function (ply, request)
    if not IsValid (ply) or not ply:IsAdmin () then return end   -- always permission-check admin features

    local dump = {
        GeneratedAt = os.time (),
        RequestedBy = ply:SteamID64 (),
        ConCommands = request.IncludeConCommands and ChrononLabsStreamNetSafeCopy (concommand.GetTable ()) or nil,
        Hooks       = request.IncludeHooks       and ChrononLabsStreamNetSafeCopy (hook.GetTable ())       or nil
    }

    ChrononLabsStreamNet.SendRaw ("ExampleDebugDump", ply, util.TableToJSON (dump, false), {
        ChunkSize = 16384, Compress = true, CompressAt = 512,
        RetryInterval = 0.75, Timeout = 30, MaximumRetries = 20, Window = 8, ReliableData = false,
        OnComplete = function (ok, reason, transfer)
            print ("[DebugDump] sent:", ok, reason, "raw:", transfer.RawSize, "packed:", transfer.PackedSize)
        end
    })
end)
```

```lua
-- Client
ChrononLabsStreamNet.Receive ("ExampleDebugDump", function (json)
    local dump = util.JSONToTable (json)
    if not dump then print ("Failed to decode debug dump") return end

    print ("ConCommand entries:", dump.ConCommands and table.Count (dump.ConCommands) or 0)
    print ("Hook entries:",       dump.Hooks       and table.Count (dump.Hooks)       or 0)
end)
```

## License

MIT. You may use, modify, distribute, and include it in your own projects, including commercial addons, as long as the original copyright and license notice are preserved.

If the file is renamed, merged, or implemented directly inside another addon, please keep clear credit to ChrononLabs-StreamNet / ChrononLabs in the source or documentation.

## Contributions

Contributions are welcome. Bug reports, fixes, examples, docs improvements, and real-world test cases are all useful.
