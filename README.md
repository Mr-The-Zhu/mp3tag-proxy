# Mp3tag Proxy

Fixes **403 Forbidden** errors when using Mp3tag tag-source scripts with
Beatport and Traxsource, caused by Cloudflare bot protection.

The proxy runs locally on your machine and sits between Mp3tag and the music
platforms. It appears as a small icon in the system tray.

---

## Requirements

- Windows 10 / 11
- [Mp3tag v3.26+](https://www.mp3tag.de/)
- Tag-source scripts for Beatport or Traxsource (see below)

---

## Installation

### 1. Download

Get `mp3tag_proxy.exe` from the [Releases](../../releases/latest) page.  
Place it in any folder (e.g. `C:\Tools\mp3tag_proxy\`).

### 2. Install your tag-source scripts

**Beatport** — stevehero's Beatport v6 scripts:  
[community.mp3tag.de › WS Beatport by stevehero](https://community.mp3tag.de/t/ws-beatport-com-by-stevehero-release-single-track-artwork-tagging/12568)

**Traxsource / Beatport / SoundCloud** — scripts by Jordi & Jean-Claude:  
[community.mp3tag.de › WS Beatport · Traxsource · SoundCloud](https://community.mp3tag.de/t/ws-beatport-traxsource-soundcloud-updated-fixed-scripts-2026-by-jordi-claude/71123)

Place all script files into Mp3tag's sources folder:
```
%APPDATA%\Mp3tag\data\sources\
```

### 3. Edit the script files

Apply the small edits described in [MODIFICATIONS.md](MODIFICATIONS.md) to route
requests through the proxy.

> **Important:** save edited files as **UTF-8 without BOM** (Notepad++ or VS Code).
> Do not edit `.settings` files.

### 4. Restart Mp3tag

**File → Exit**, then reopen — so Mp3tag reloads the updated scripts.

---

## Daily use

1. Double-click `mp3tag_proxy.exe`. A tray icon appears near the clock.
2. Use Mp3tag tag sources as usual — tags are fetched via the proxy.
3. Right-click the tray icon → **Quit** when done, or leave it running
   (auto-exits after 30 minutes of inactivity).

---

## Tray menu

| Item | Action |
|---|---|
| Auto-exit when idle (30 min) | Toggle idle auto-exit on/off |
| Open log | Opens `proxy.log` in your text editor |
| About | Shows version info |
| Quit | Shuts down the proxy |

---

## Troubleshooting

**403 Forbidden** — proxy isn't running. Double-click `mp3tag_proxy.exe`.
Also check that no VPN is blocking `127.0.0.1`.

**Connection error / no tray icon** — proxy failed to start. Check `proxy.log`
next to the `.exe`. Port 8787 may be in use by another application.

**No results in Mp3tag** — fully exit Mp3tag (File → Exit) and reopen.
Check that the script files are in `%APPDATA%\Mp3tag\data\sources\`.

**Tray icon not visible** — click the `^` arrow on the taskbar near the clock.

---

## Contact

Questions, issues, or requests — find me on the Mp3tag community forum as **The_Zhu**.
