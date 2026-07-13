# Mp3tag Proxy

Fixes **403 Forbidden** errors when using Mp3tag tag-source scripts with
Beatport and Traxsource, caused by Cloudflare bot protection.

The proxy runs locally on your machine and sits between Mp3tag and the music
platforms. It appears as a small icon in the system tray.

---

## Requirements

- Windows 10 / 11 with a Chromium browser — Microsoft Edge (preinstalled),
  Google Chrome, or Brave — used to clear Beatport's Cloudflare challenge
- [Mp3tag](https://www.mp3tag.de/) — the version your scripts require
  (stevehero's Beatport v6: **v3.22+**, Traxsource: **v3.26+**)
- Tag-source scripts for Beatport or Traxsource (see below)

---

## Installation

### 1. Download

Get `mp3tag_proxy.exe` from the [Releases](../../releases/latest) page.  
Place it in any folder (e.g. `C:\Tools\mp3tag_proxy\`).

### 2. Install your tag-source scripts

**Beatport** — stevehero's Beatport v6 scripts:  
[community.mp3tag.de › WS Beatport by stevehero](https://community.mp3tag.de/t/ws-beatport-com-by-stevehero-release-single-track-artwork-tagging/12568)

Beatport · **Traxsource** · SoundCloud — Updated & Fixed Scripts 2026 (by Jordi & Claude):  
[community.mp3tag.de › WS Beatport · Traxsource · SoundCloud](https://community.mp3tag.de/t/ws-beatport-traxsource-soundcloud-updated-fixed-scripts-2026-by-jordi-claude/71123)

Place all script files into Mp3tag's sources folder:
```
%APPDATA%\Mp3tag\data\sources\
```

### 3. Edit the script files

Apply the small edits described in [MODIFICATIONS.md](MODIFICATIONS.md) to route
requests through the proxy.

> **Important:** do not edit `.settings` files.

### 4. Restart Mp3tag

**File → Exit**, then reopen — so Mp3tag reloads the updated scripts.

---

## Daily use

1. Double-click `mp3tag_proxy.exe`. A tray icon appears near the clock.
2. Use Mp3tag tag sources as usual — tags are fetched via the proxy.
3. Right-click the tray icon → **Quit** when done, or leave it running
   (auto-exits after 30 minutes of inactivity).

> **Beatport note:** Beatport is behind a Cloudflare challenge that needs a real
> browser to pass. The first Beatport request (and occasionally later, when the
> clearance expires) takes ~5–10 seconds while the proxy quietly uses a Chromium
> browser (Edge, Chrome, or Brave) in the background to clear it — the tray tooltip
> shows *"solving challenge…"* meanwhile. Everything after that is instant, and the
> brief browser process closes itself. **Traxsource is unaffected.**

---

## Tray menu

| Item | Action |
|---|---|
| Auto-exit when idle (30 min) | Toggle idle auto-exit on/off |
| Check for updates | Toggle the startup version check |
| Open log | Opens `proxy.log` in your text editor |
| About | Shows version info |
| Quit | Shuts down the proxy |

> On startup the proxy checks GitHub for a newer release (version numbers only —
> nothing personal is sent). If one is available, the About splash shows it and the
> tray gets an **Update available** item. Turn the check off with the
> **Check for updates** toggle.

---

## Troubleshooting

**403 Forbidden** — proxy isn't running. Double-click `mp3tag_proxy.exe`.
Also check that no VPN is blocking `127.0.0.1`.

**Beatport is slow on the first request** — that's expected. The proxy is using
Edge in the background to clear Cloudflare's challenge (~5–10 s); it's cached
afterwards, so following requests are fast.

**Connection error / no tray icon** — proxy failed to start. Check `proxy.log`
next to the `.exe`. Port 8787 may be in use by another application.

**No results in Mp3tag** — fully exit Mp3tag (File → Exit) and reopen.
Check that the script files are in `%APPDATA%\Mp3tag\data\sources\`.

**Tray icon not visible** — click the `^` arrow on the taskbar near the clock.

---

## Contact

Questions, issues, or requests — find me on the Mp3tag community forum as **The_Zhu**.

---

## Disclaimer

Mp3tag Proxy is an independent, third-party tool and is not affiliated with or
endorsed by Mp3tag. Mp3tag is a registered trademark and the exclusive property
of its holder, as is its logo.
