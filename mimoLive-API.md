# mimoLive API Reference for Claude

> **How to use this file:** Drop it into your project's `CLAUDE.md`, `.claude/` directory, or paste it into a Claude conversation when building apps that talk to mimoLive. Claude will then know every endpoint, gotcha, and pattern needed to write correct API calls.

mimoLive is a professional live video production app for macOS. It exposes a local HTTP API (JSON:API format) and a WebSocket for real-time events. There is also an optional companion app called **mlController** that provides a simplified proxy API.

---

## Quick Start

```bash
# Check if mimoLive is running and has open documents
curl http://localhost:8989/api/v1/documents

# Start a show
curl http://localhost:8989/api/v1/documents/{DocID}/setLive

# Toggle a layer on/off
curl http://localhost:8989/api/v1/documents/{DocID}/layers/{LayerID}/toggleLive
```

---

## mimoLive API (port 8989)

**Base URL:** `http://localhost:8989/api/v1`

- **Format:** [JSON:API](https://jsonapi.org/) — responses have `data`, `links`, `relationships`, and optionally `included`
- **Auth:** None by default. If enabled in Preferences > Remote Control, use header `X-MimoLive-Password-SHA256: <hex>` or query param `?pwSHA256=<hex>` (SHA-256 of the UTF-8 password)
- **Content-Type for writes:** `application/vnd.api+json` for PUT/PATCH requests
- **Network access:** Accessible on `localhost` and on the local network via `.local` hostname

### Documents

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/documents` | List all open documents |
| GET | `/documents/{DocID}` | Single document (includes sideloaded layers, sources, output-destinations) |
| GET | `/documents/{DocID}/programOut` | Current program output as an image |
| GET\|POST | `/documents/{DocID}/setLive` | Start the show |
| GET\|POST | `/documents/{DocID}/setOff` | Stop the show |
| GET\|POST | `/documents/{DocID}/toggleLive` | Toggle show on/off |

**Document attributes:**
```json
{
  "name": "My Show.tvshow",
  "live-state": "off",
  "duration": 0,
  "formatted-duration": "00:00:00",
  "show-start": null,
  "programOutputMasterVolume": 1.0,
  "metadata": {
    "title": "My Show",
    "comments": "...",
    "author": "...",
    "show": "...",
    "width": 1920,
    "height": 1080,
    "framerate": 30,
    "samplerate": 48000,
    "duration": 0
  },
  "outputs": [
    { "id": "record", "type": "record", "live-state": "off" },
    { "id": "stream", "type": "stream", "live-state": "off" },
    { "id": "playout", "type": "playout", "live-state": "off" },
    { "id": "fullscreen", "type": "fullscreen", "live-state": "off" }
  ]
}
```

**Document relationships:** `sources`, `layers`, `output-destinations`, `layer-sets`

### Layers

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/documents/{DocID}/layers` | List all layers in a document |
| GET | `/documents/{DocID}/layers/{LayerID}` | Single layer |
| PUT | `/documents/{DocID}/layers/{LayerID}` | Modify layer attributes (omit unchanged values) |
| GET\|POST | `.../layers/{LayerID}/setLive` | Switch layer live |
| GET\|POST | `.../layers/{LayerID}/setOff` | Switch layer off |
| GET\|POST | `.../layers/{LayerID}/toggleLive` | Toggle layer on/off |
| GET\|POST | `.../layers/{LayerID}/signals/{SignalID}` | Trigger a signal on a layer |
| GET\|POST | `.../layers/{LayerID}/cycleThroughVariants` | Cycle to next variant |
| GET\|POST | `.../layers/{LayerID}/cycleThroughVariantsBackwards` | Cycle to previous variant |
| GET\|POST | `.../layers/{LayerID}/setLiveFirstVariant` | Activate first variant |
| GET\|POST | `.../layers/{LayerID}/setLiveLastVariant` | Activate last variant |
| GET\|POST | `.../layers/{LayerID}/inputs/{SourceInputKey}/mediacontrol/{Command}` | Media playback control |

**Layer attributes:**
```json
{
  "name": "Lower Third",
  "live-state": "off",
  "volume": null,
  "composition-id": "com.boinx.layer.lowerThird",
  "index": 5,
  "input-values": { "tvIn_Title": "John Doe", "tvIn_Subtitle": "CEO" },
  "input-descriptions": { "tvIn_Title": { "uiName": "Title", "type": "string", "group-name": "Content", ... } },
  "output-values": { "tvOut_SettingName": "Lower Third (◹ Dissolve)", "tvOut_Opaque": false }
}
```

**Layer relationships:** `variants`, `live-variant`, `active-variant`, `document`

**Modifying a layer (PUT):** Only include the fields you want to change:
```bash
curl -X PUT \
  -H "Content-Type: application/vnd.api+json" \
  -d '{"input-values": {"tvIn_Title": "Jane Smith"}}' \
  http://localhost:8989/api/v1/documents/{DocID}/layers/{LayerID}
```

### Variants

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `.../layers/{LayerID}/variants` | List all variants for a layer |
| GET | `.../variants/{VariantID}` | Single variant |
| PUT | `.../variants/{VariantID}` | Modify variant |
| GET\|POST | `.../variants/{VariantID}/setLive` | Activate variant (also makes layer live) |
| GET\|POST | `.../variants/{VariantID}/setOff` | Deactivate variant (also turns layer off) |
| GET\|POST | `.../variants/{VariantID}/toggleLive` | Toggle variant |

**Variant attributes:** Same as layer (`live-state`, `input-descriptions`, `input-values`). Relationships: `layer`.

### Sources

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/documents/{DocID}/sources` | List all sources in a document |
| GET | `.../sources/{SourceID}` | Single source (includes sideloaded filters) |
| PUT | `.../sources/{SourceID}` | Modify source attributes |
| GET | `.../sources/{SourceID}/preview` | Source preview image |
| GET\|POST | `.../sources/{SourceID}/mediacontrol/{Command}` | Media playback control |
| GET\|POST | `.../sources/{SourceID}/signals/{SignalID}` | Trigger signal on source |

**Common source attributes (all types):**
```json
{
  "name": "Camera 1",
  "source-type": "com.boinx.mimoLive.sources.deviceVideoSource",
  "summary": "MacBook Air Camera",
  "audio": false,
  "video": true,
  "gain": 1.0,
  "tally-state": "off",
  "is-hidden": false,
  "is-static": false
}
```

**Source ID format:** `{DocID}-{UUID}` — the document ID is prefixed to the source UUID.

**Source relationships:** `filters`, `document`

**Known source types:**
| source-type | Description | Extra attributes |
|-------------|-------------|-----------------|
| `com.boinx.mimoLive.sources.deviceVideoSource` | Camera/capture device | `video-device-connected` |
| `com.boinx.mimoLive.sources.imageSource` | Static image | `filepath` |
| `com.boinx.mimoLive.sources.zoomparticipant` | Zoom meeting participant | `zoom-userid`, `zoom-username`, `zoom-userselectiontype`, `zoom-videoresolution` |
| `com.boinx.mimoLive.sources.socialSource` | Social/streaming source | (none extra) |
| `com.boinx.boinxtv.source.placeholder` | Placeholder source | `composition-id`, `input-values`, `input-descriptions`, `output-values` |
| `com.boinx.boinxtv.source.sports.teamdata` | Sports team data | `composition-id`, `input-values`, `input-descriptions`, `output-values` |

**Tally states:** `off`, `preview`, `program`, `in-use`

### Filters

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `.../sources/{SourceID}/filters` | List filters on a source |
| GET | `.../filters/{FilterID}` | Single filter |
| PUT | `.../filters/{FilterID}` | Modify filter |
| GET\|POST | `.../filters/{FilterID}/signals/{SignalID}` | Trigger signal on filter |

### Output Destinations

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/documents/{DocID}/output-destinations` | List output destinations |
| GET | `.../output-destinations/{OutputID}` | Single output destination |
| PUT | `.../output-destinations/{OutputID}` | Modify output destination settings |
| GET\|POST | `.../output-destinations/{OutputID}/setLive` | Start output |
| GET\|POST | `.../output-destinations/{OutputID}/setOff` | Stop output |
| GET\|POST | `.../output-destinations/{OutputID}/toggleLive` | Toggle output |

**Output destination attributes:**
```json
{
  "title": "File Recording",
  "type": "File Recording",
  "live-state": "off",
  "ready-to-go-live": true,
  "starts-with-show": true,
  "stops-with-show": true,
  "summary": "~/Movies, Program Output (H.264), Program Mix (AAC)",
  "settings": {
    "location": "~/Movies",
    "filename": "%show %year-%month-%day %hour-%minute-%second.%extension"
  }
}
```

**Output destination types:** File Recording, Live Streaming (RTMP), NDI, Fullscreen

**Modifying RTMP streaming settings:**
```bash
curl -X PUT \
  -H "Content-Type: application/vnd.api+json" \
  -d '{"data": {"attributes": {"settings": {"rtmpurl": "rtmp://stream.example.com/live", "streamingkey": "your-key"}}}}' \
  http://localhost:8989/api/v1/documents/{DocID}/output-destinations/{OutputID}
```

### Layer Sets

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/documents/{DocID}/layer-sets` | List layer sets |
| GET | `.../layer-sets/{LayerSetID}` | Single layer set |
| GET\|POST | `.../layer-sets/{LayerSetID}/recall` | Recall/activate a layer set |

Layer sets allow you to set the live state of multiple layers at once.

### Data Stores (since mimoLive 6.8)

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/documents/{DocID}/datastores` | List data stores |
| GET | `.../datastores/{DataStoreID}` | Single data store (includes rows) |
| GET | `.../datastores/{DataStoreID}/rows` | List rows |
| GET\|POST | `.../rows/{RowID}/focus` | Focus a row |
| GET\|POST | `.../rows/{RowID}/unfocus` | Unfocus a row |
| GET\|POST | `.../rows/{RowID}/toggleFocus` | Toggle row focus |
| GET\|POST | `.../rows/{RowID}/signal` | Trigger signal on a row |

### Devices

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/devices` | List all available video/audio devices |
| GET | `/devices/{DeviceID}` | Single device |

**Device attributes:**
```json
{
  "name": "MacBook Air Camera",
  "audio": false,
  "video": true,
  "connected": true,
  "device-type": "com.boinx.devicetype.avfoundation",
  "tally-state": "off"
}
```

**Device types:** `com.boinx.devicetype.avfoundation` (local), `com.boinx.devicetype.ndi` (NDI network)

---

## Zoom Meeting Endpoints

These endpoints control Zoom meetings integrated into mimoLive via the Zoom plugin. They are separate from the document/source endpoints.

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET\|POST | `/zoom/join` | Join a meeting |
| GET\|POST | `/zoom/leave` | Leave the meeting |
| GET\|POST | `/zoom/end` | End/terminate the meeting (host only) |
| GET\|POST | `/zoom/participants` | List meeting participants |
| GET | `/zoom/meetingaction?command={cmd}` | Execute a meeting action |
| GET | `/zoom/meetingaction?command={cmd}&userid={id}` | Execute per-user action |

### Join Meeting

Query parameters (passed as URL params, not JSON body):
```
meetingid=123456789        # Required
passcode=abc123            # Optional
displayname=mimoLive       # Optional
zoomaccountname=MyAccount  # Optional
virtualcamera=true         # Optional (boolean)
webinartoken=...           # Optional
```

Example:
```bash
curl "http://localhost:8989/api/v1/zoom/join?meetingid=123456789&displayname=mimoLive&virtualcamera=true"
```

### Participants Response

```json
{
  "data": [
    {
      "id": 16786432,
      "name": "John Doe",
      "userRole": "Host",
      "isHost": true,
      "isCoHost": false,
      "isVideoOn": true,
      "isAudioOn": true,
      "isTalking": false,
      "isRaisingHand": false
    }
  ]
}
```

**Note:** When not in a meeting, `data` is an empty array `[]`.

### Zoom Source Assignment

Zoom participant video feeds appear as **sources** in the document (type `com.boinx.mimoLive.sources.zoomparticipant`). To assign a participant to a source, PATCH the source:

```bash
# Assign specific participant (selectionType is inferred as 1)
curl -X PATCH \
  -H "Content-Type: application/vnd.api+json" \
  -d '{"zoom-userid": 16786432}' \
  http://localhost:8989/api/v1/sources/{SourceID}

# Set to automatic (next active speaker)
curl -X PATCH \
  -H "Content-Type: application/vnd.api+json" \
  -d '{"zoom-userselectiontype": 2}' \
  http://localhost:8989/api/v1/sources/{SourceID}

# Set to screen share
curl -X PATCH \
  -H "Content-Type: application/vnd.api+json" \
  -d '{"zoom-userselectiontype": 6}' \
  http://localhost:8989/api/v1/sources/{SourceID}
```

**Zoom selection types:**
| Value | Meaning |
|-------|---------|
| 0 | Unassigned |
| 1 | Specific participant (set via `zoom-userid`) |
| 2 | Automatic (next active speaker) |
| 6 | Screen share |

### Meeting Action Commands

**Per-user** (require `&userid={id}`):
- `muteVideo` / `unmuteVideo`
- `muteAudio` / `unmuteAudio`

**Meeting-wide** (host/co-host only):
- `requestRecordingPermission`
- `muteAll` / `unmuteAll`
- `enableUnmuteBySelf` / `disableUnmuteBySelf`
- `lockMeeting` / `unlockMeeting`
- `lowerAllHands`
- `allowParticipantsToChat` / `disallowParticipantsToChat`
- `allowParticipantsToShare` / `disallowParticipantsToShare`
- `allowParticipantsToStartVideo` / `disallowParticipantsToStartVideo`
- `allowParticipantsToShareWhiteBoard` / `disallowParticipantsToShareWhiteBoard`
- `enableAutoAllowLocalRecordingRequest` / `disableAutoAllowLocalRecordingRequest`
- `allowParticipantsToRename` / `disallowParticipantsToRename`
- `showParticipantProfilePictures` / `hideParticipantProfilePictures`

**Sharing/VoIP:**
- `shareFitWindowMode` / `pauseShare` / `resumeShare`
- `joinVoip` / `leaveVoip`

Example:
```bash
# Mute a specific participant's audio
curl "http://localhost:8989/api/v1/zoom/meetingaction?command=muteAudio&userid=16786432"

# Mute all participants
curl "http://localhost:8989/api/v1/zoom/meetingaction?command=muteAll"
```

---

## WebSocket

**Endpoint:** `ws://localhost:8989/api/v1/socket`

The WebSocket pushes real-time state change events so you don't need to poll.

### Connection

```javascript
const ws = new WebSocket("ws://localhost:8989/api/v1/socket");

// MUST send ping every 5 seconds or connection times out at 15s
const pingInterval = setInterval(() => {
  ws.send(JSON.stringify({ event: "ping" }));
}, 5000);

ws.onmessage = (event) => {
  const msg = JSON.parse(event.data);
  if (msg.event === "ping" || msg.event === "pong") return; // ignore keepalive

  // Handle state changes
  console.log(msg.event, msg.type, msg);
};

ws.onclose = () => {
  clearInterval(pingInterval);
  // Reconnect after 2 seconds
  setTimeout(connect, 2000);
};
```

### Event Messages

```json
{"event": "ping"}
{"event": "pong"}
{"event": "added", "type": "documents", ...}
{"event": "removed", "type": "documents", ...}
{"event": "changed", "type": "sources", ...}
{"event": "changed", "type": "layers", ...}
```

**Event types:** `added`, `removed`, `changed`
**Resource types:** `documents`, `sources`, `layers`, `variants`, `output-destinations`, etc.

**Best practice:** On any `added`/`removed`/`changed` event, re-fetch the relevant resource to get the full updated state.

---

## Sparse Filtering

Reduce API response size with query parameters:

```bash
# Only include specific attributes for a type
curl "http://localhost:8989/api/v1/documents?fields[documents]=name,live-state"

# Control which relationships are included
curl "http://localhost:8989/api/v1/documents?include=layers,sources"
```

---

## Live State Values

Used across documents, layers, variants, and output destinations:

| Value | Meaning |
|-------|---------|
| `off` | Not active |
| `live` | Currently active/on air |
| `preview` | In preview mode (may go live) |

---

## Important Gotchas

1. **PATCH uses short source path** — `PATCH /api/v1/sources/{SourceID}` works, NOT necessarily nested under `/documents/{DocID}/sources/...`
2. **Content-Type matters** — PUT/PATCH requests need `Content-Type: application/vnd.api+json`
3. **Zoom endpoints use GET** — `zoom/leave`, `zoom/join`, `zoom/end` all accept GET (not just POST)
4. **Meeting actions are GET** — `zoom/meetingaction?command=...` is a GET request
5. **Zoom user IDs are numeric** — e.g., `16786432`, not strings
6. **409 when not in a meeting** — ALL `meetingaction` commands return 409 Conflict when not in a meeting
7. **No meeting settings query** — There's no endpoint to read current meeting-wide settings (mute state, lock state, etc.). Actions are fire-and-forget only. Per-participant state is available via `/zoom/participants`
8. **Source IDs include DocID** — Source IDs are formatted as `{DocID}-{UUID}`
9. **Username resolution is async** — After assigning a Zoom participant to a source, the `zoom-username` updates asynchronously (~500ms). Re-fetch to see the updated name
10. **Layer/source IDs are stable** — As long as you don't delete and recreate them, IDs persist across sessions. Reordering layers/sources does not change IDs
11. **Right-click for API URLs** — In the mimoLive UI, right-click any element and select "Copy API Endpoint" to get its exact URL
12. **Enable HTTP Server** — The API must be enabled in mimoLive's Preferences > Remote Control

---

## mlController Proxy API (port 8990)

[mlController](https://github.com/boinx/mlController) is an open-source macOS menu bar app that monitors and controls mimoLive. It provides a simplified proxy API on port 8990 with optional authentication.

### Authentication

Optional HTTP Basic Auth or custom header:
- `Authorization: Basic base64(username:password)`
- `x-mlcontroller-password: <password>`
- If enabled, returns `401 Unauthorized` with `WWW-Authenticate` header

### Status & Control

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/status` | Get mimoLive status (running, documents, etc.) |
| POST | `/api/start` | Launch mimoLive |
| POST | `/api/stop` | Quit mimoLive |
| POST | `/api/restart` | Restart mimoLive |
| POST | `/api/open` | Open a document (body: `{"path": "/path/to/file.tvshow"}`) |
| POST | `/api/select` | Select mimoLive version (body: `{"path": "/Applications/mimoLive.app"}`) |
| GET | `/api/docicon` | Get .tvshow file icon as PNG (cached 1 hour) |

**Status response:**
```json
{
  "running": true,
  "openDocuments": [
    { "id": "1536240957", "name": "My Show", "path": "/path/to/show.tvshow" }
  ],
  "localDocuments": ["/Users/me/Documents/show1.tvshow", "/Users/me/Documents/show2.tvshow"],
  "selectedMimoLive": "mimoLive (6.15)",
  "selectedMimoLivePath": "/Applications/mimoLive.app",
  "availableMimoLiveApps": [
    { "name": "mimoLive (6.15)", "path": "/Applications/mimoLive.app" }
  ]
}
```

### Zoom Proxy Endpoints

mlController wraps mimoLive's Zoom endpoints into a simpler JSON interface:

| Method | Endpoint | Body | Description |
|--------|----------|------|-------------|
| GET | `/api/zoom/sources` | — | List Zoom video sources with attributes |
| GET | `/api/zoom/participants` | — | List current Zoom participants |
| POST | `/api/zoom/assign` | `{"sourceId", "selectionType", "userId"}` | Assign participant to source |
| POST | `/api/zoom/join` | `{"meetingId", "displayName", "passcode?", "zoomAccountName?", "virtualCamera?"}` | Join a meeting |
| POST | `/api/zoom/leave` | — | Leave current meeting |
| POST | `/api/zoom/request-recording` | — | Request recording permission from host |

### WebSocket (mlController)

**Endpoint:** `ws://localhost:8990/ws`

- Sends current status snapshot immediately on connect
- Pushes updated status JSON whenever state changes
- Same format as `GET /api/status`
- Fallback: poll `/api/status` every 2 seconds if WebSocket unavailable

---

## Common Recipes

### List all layers and their states
```bash
curl -s http://localhost:8989/api/v1/documents/{DocID}/layers | \
  python3 -c "
import json, sys
for l in json.load(sys.stdin)['data']:
    a = l['attributes']
    name = a.get('output-values', {}).get('tvOut_SettingName', a.get('name', '?'))
    print(f\"{l['id']}: {name} [{a['live-state']}]\")
"
```

### Toggle a layer by name
```bash
# First find the layer ID
LAYER_ID=$(curl -s http://localhost:8989/api/v1/documents/{DocID}/layers | \
  python3 -c "
import json, sys
for l in json.load(sys.stdin)['data']:
    if 'Lower Third' in l['attributes'].get('output-values', {}).get('tvOut_SettingName', ''):
        print(l['id']); break
")

# Then toggle it
curl http://localhost:8989/api/v1/documents/{DocID}/layers/$LAYER_ID/toggleLive
```

### Update lower third text and go live
```bash
# Update the text
curl -X PUT \
  -H "Content-Type: application/vnd.api+json" \
  -d '{"input-values": {"tvIn_Title": "Jane Smith", "tvIn_Subtitle": "CEO"}}' \
  http://localhost:8989/api/v1/documents/{DocID}/layers/{LayerID}

# Go live
curl http://localhost:8989/api/v1/documents/{DocID}/layers/{LayerID}/setLive
```

### Cycle through layer variants
```bash
curl http://localhost:8989/api/v1/documents/{DocID}/layers/{LayerID}/cycleThroughVariants
```

### Start recording
```bash
# Find the File Recording output destination
OUTPUT_ID=$(curl -s http://localhost:8989/api/v1/documents/{DocID}/output-destinations | \
  python3 -c "
import json, sys
for o in json.load(sys.stdin)['data']:
    if o['attributes']['type'] == 'File Recording':
        print(o['id']); break
")

# Start it
curl http://localhost:8989/api/v1/documents/{DocID}/output-destinations/$OUTPUT_ID/setLive
```

### Start RTMP streaming
```bash
# Configure stream URL and key
curl -X PUT \
  -H "Content-Type: application/vnd.api+json" \
  -d '{"data": {"attributes": {"settings": {"rtmpurl": "rtmp://stream.example.com/live", "streamingkey": "your-key"}}}}' \
  http://localhost:8989/api/v1/documents/{DocID}/output-destinations/{OutputID}

# Start streaming
curl http://localhost:8989/api/v1/documents/{DocID}/output-destinations/{OutputID}/setLive
```

### Monitor mimoLive state in real-time (Node.js)
```javascript
const WebSocket = require("ws");

function connect() {
  const ws = new WebSocket("ws://localhost:8989/api/v1/socket");

  const ping = setInterval(() => ws.send(JSON.stringify({ event: "ping" })), 5000);

  ws.on("message", (data) => {
    const msg = JSON.parse(data);
    if (msg.event === "ping" || msg.event === "pong") return;
    console.log(`[${msg.event}] ${msg.type || "unknown"}`, msg);
  });

  ws.on("close", () => {
    clearInterval(ping);
    setTimeout(connect, 2000);
  });

  ws.on("error", (err) => {
    console.error("WebSocket error:", err.message);
    ws.close();
  });
}

connect();
```

### Join Zoom meeting and assign participants
```bash
# Join meeting
curl "http://localhost:8989/api/v1/zoom/join?meetingid=123456789&displayname=mimoLive&virtualcamera=true"

# Wait a moment for participants to appear, then list them
sleep 3
curl http://localhost:8989/api/v1/zoom/participants

# Find Zoom sources in the document
curl http://localhost:8989/api/v1/documents/{DocID}/sources | \
  python3 -c "
import json, sys
for s in json.load(sys.stdin)['data']:
    if s['attributes'].get('source-type') == 'com.boinx.mimoLive.sources.zoomparticipant':
        a = s['attributes']
        print(f\"{s['id']}: {a['name']} -> {a.get('zoom-username', 'unassigned')} (type={a.get('zoom-userselectiontype', '?')})\")
"

# Assign participant 16786432 to a Zoom source
curl -X PATCH \
  -H "Content-Type: application/vnd.api+json" \
  -d '{"zoom-userid": 16786432}' \
  http://localhost:8989/api/v1/sources/{SourceID}
```
