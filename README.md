# mimoLive API Reference

A comprehensive reference for the [mimoLive](https://mimolive.com) HTTP API and WebSocket protocol, covering every endpoint, response format, and common recipe — verified against a live mimoLive instance.

This reference is designed to work with **AI coding assistants** like [Claude](https://claude.ai), [Cursor](https://cursor.com), and [GitHub Copilot](https://github.com/features/copilot). Drop it into your project context and your AI assistant will know exactly how to write correct mimoLive API calls.

## What's Covered

- **Documents** — list, start/stop shows, get program output
- **Layers & Variants** — create/delete layers, toggle live, modify parameters, cycle variants, trigger signals
- **Sources** — create/delete sources, modify, preview images, media control
- **Output Destinations** — create/delete, file recording, RTMP streaming, NDI, fullscreen, and more
- **Layer Sets** — create, update, delete, and recall preset layer configurations
- **Data Stores** — focus/unfocus rows, trigger signals
- **Devices** — list available video/audio devices
- **Zoom Meetings** — join/leave, participants, source assignment, 20+ meeting actions
- **WebSocket** — real-time state change events with connection examples
- **mlController Proxy** — simplified API on port 8990 with optional auth

## Quick Start

### With Claude Code

Copy the reference into your project:

```bash
# Option A: Add to your .claude directory
mkdir -p .claude
cp mimoLive-API.md .claude/

# Option B: Reference in your CLAUDE.md
echo "For mimoLive API details, see mimoLive-API.md" >> CLAUDE.md
```

Then ask Claude to help you build:

```
Help me write a script that toggles the lower third layer when I press a hotkey
```

### With other AI assistants

Paste the contents of [`mimoLive-API.md`](mimoLive-API.md) into your conversation or project context.

### With curl

The reference includes working curl examples. Here's a taste:

```bash
# List open documents
curl http://localhost:8989/api/v1/documents

# Start the show
curl http://localhost:8989/api/v1/documents/{DocID}/setLive

# Toggle a layer
curl http://localhost:8989/api/v1/documents/{DocID}/layers/{LayerID}/toggleLive

# Create a live-streaming output destination
curl -X POST -H "Content-Type: application/vnd.api+json" \
  -d '{"data": {"attributes": {"output-destination-type": "com.boinx.mimoLive.outputDestination.liveStreaming", "settings": {"rtmpurl": "rtmp://...", "streamingkey": "your-key"}}}}' \
  http://localhost:8989/api/v1/documents/{DocID}/output-destinations

# Join a Zoom meeting
curl "http://localhost:8989/api/v1/zoom/join?meetingid=123456789&displayname=mimoLive"
```

## Using with mlController

[mlController](https://github.com/boinx/mlController) is an open-source macOS menu bar app that provides a simplified proxy API for mimoLive on port 8990. The reference documents both the native mimoLive API and the mlController proxy endpoints.

If you're using Claude Code with the mlController repo, the `/mimolive` slash command loads this reference automatically.

## Prerequisites

- [mimoLive](https://mimolive.com) installed and running on macOS
- HTTP Server enabled in mimoLive Preferences > Remote Control
- API accessible at `http://localhost:8989/api/v1`

## Official Documentation

- [mimoLive HTTP API](https://mimolive.com/user-manual/remote-control-automation/http-api/)
- [mimoLive Endpoints](https://mimolive.com/user-manual/customization/http-api/endpoints/)
- [mimoLive Data Types](https://mimolive.com/user-manual/customization/http-api/data-types/)

## Community

Join the **mimoLive Experts** community on Discord:

**[Join the Discord](https://mee6.xyz/i/2MzHLgRE4w)**

## License

This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.
