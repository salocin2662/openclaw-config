---
name: spotify-player
description: Control Spotify playback and search via the spogo CLI. Use for playing tracks, pausing, skipping, searching library, managing devices, and viewing current playback status.
homepage: https://www.spotify.com
metadata:
  {
    "openclaw":
      {
        "emoji": "🎵",
        "requires": { "anyBins": ["spogo"] },
        "install":
          [
            {
              "id": "brew",
              "kind": "brew",
              "formula": "spogo",
              "tap": "steipete/tap",
              "bins": ["spogo"],
              "label": "Install spogo (brew)",
            },
          ],
      },
  }
---

# spogo

Use `spogo` for Spotify playback/search on Linux/WSL2.

Requirements

- Spotify Premium account (required for all playback features)
- `spogo` installed and authenticated

Setup

1. Install spogo binary to ~/.local/bin/spogo
2. Authenticate: `spogo auth import --browser chrome` (or `spogo auth paste` for manual token entry)

Common Commands

- Search: `spogo search track "query"` | `spogo search artist "name"` | `spogo search playlist "name"`
- Playback: `spogo play` | `spogo pause` | `spogo next` | `spogo prev`
- Status: `spogo status`
- Devices: `spogo device list` | `spogo device set "<device-name>"`
- Library: `spogo library tracks` | `spogo library playlists`
- Playlists: `spogo playlist list` | `spogo playlist tracks "<playlist-name>"`

Notes

- Config stored at ~/.config/spogo/
- Run `spogo --help` for full command list
- For Spotify Connect (web playback): ensure Spotify is open on a device, then `spogo device set "<device>"`