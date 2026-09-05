# Mp3tag Proxy

Fixes **403 Forbidden** errors when using Mp3tag tag-source scripts with
Beatport and Traxsource, caused by Cloudflare bot protection.

The proxy runs locally on your machine and sits between Mp3tag and the music
platforms. It appears as a small icon in the system tray.

---

## Requirements

- Windows 10 / 11 with a Chromium browser: Microsoft Edge (preinstalled),
  Google Chrome, or Brave, used to clear Beatport's Cloudflare challenge
- [Mp3tag](https://www.mp3tag.de/), the version your scripts require
  (stevehero's Beatport v6: **v3.22+**, Traxsource: **v3.26+**)
- Tag-source scripts for Beatport or Traxsource (see below)

---

## Installation

### 1. Download

Get `mp3tag_proxy.exe` from the [Releases](../../releases/latest) page.
If you use Beatport, also grab `beatport_scripts_patcher.exe` from the same page.
Place them in any folder (e.g. `C:\Tools\mp3tag_proxy\`).

The two ship as one release, so update them together. If you keep them side
by side, the proxy checks this for you and says so when the patcher is
behind. To see the version yourself: right click either file, Properties,
Details, or run `beatport_scripts_patcher.exe --version`.

### 2. Install your tag-source scripts

**Beatport**: stevehero's Beatport v6 scripts, from his own thread:  
[community.mp3tag.de › WS Beatport by stevehero](https://community.mp3tag.de/t/ws-beatport-com-by-stevehero-release-single-track-artwork-tagging/12568)

`beatport_scripts_patcher.exe` is written against **v6.007**, the current
release in that thread. It is the only set of Beatport scripts needed here.

**Traxsource**: Jordi & Claude's scripts:  
[community.mp3tag.de › WS Beatport · Traxsource · SoundCloud](https://community.mp3tag.de/t/ws-beatport-traxsource-soundcloud-updated-fixed-scripts-2026-by-jordi-claude/71123)

That thread is titled Beatport · Traxsource · SoundCloud, but only its
Traxsource scripts are used here. Nothing in this project needs a Beatport
script from it, so if the Beatport files in that thread are unavailable, it
does not affect the proxy or the patcher: use stevehero's above.

Place all script files into Mp3tag's sources folder:
```
%APPDATA%\Mp3tag\data\sources\
```

### 3. Edit the script files

**Beatport (stevehero's scripts):** run `beatport_scripts_patcher.exe` once.
It applies every edit those scripts need (proxy routing, full-resolution
cover art, and the current Beatport JSON fix) automatically, with a backup
of each file first. The proxy itself will also tell you, on the splash
screen and in its tray menu, if it later detects these scripts need it
again.

**Traxsource (Jordi & Claude's scripts):** apply the edits described in
[MODIFICATIONS.md](MODIFICATIONS.md) by hand, there's no automated tool for
these yet.

> **Important:** do not edit `.settings` files.

### 4. Restart Mp3tag

**File → Exit**, then reopen, so Mp3tag reloads the updated scripts.

---

## Daily use

1. Double-click `mp3tag_proxy.exe`. A tray icon appears near the clock.
2. Use Mp3tag tag sources as usual. Tags are fetched via the proxy.
3. Right-click the tray icon → **Quit** when done, or leave it running
   (auto-exits after 30 minutes of inactivity).

> **Beatport note:** Beatport is behind a Cloudflare challenge that needs a real
> browser to pass. The first Beatport request (and occasionally later, when the
> clearance expires) takes ~5–10 seconds while the proxy quietly uses a Chromium
> browser (Edge, Chrome, or Brave) in the background to clear it. The tray tooltip
> shows *"solving challenge…"* meanwhile. Everything after that is instant, and the
> brief browser process closes itself. **Traxsource is unaffected.**

---

## Tray menu

| Item | Action |
|---|---|
| Auto-exit when idle (30 min) | Toggle idle auto-exit on/off |
| Faster Beatport tagging | Hand Mp3tag a much smaller Beatport track page. Off by default |
| Check for updates | Toggle the startup version check |
| Open log | Opens `proxy.log` in your text editor |
| About | Shows version info. Click anywhere on it, press Escape, or use the close mark to dismiss it |
| Quit | Shuts down the proxy |

> On startup the proxy checks GitHub for a newer release (version numbers only,
> nothing personal is sent). If one is available, the About splash shows it and the
> tray gets an **Update available** item. Turn the check off with the
> **Check for updates** toggle.

> The proxy also checks stevehero's Beatport scripts (if installed) on
> startup. If they need an edit, a red line appears on the splash and the tray
> gets a **Fix Beatport scripts…** item. Clicking that line, or that item, runs
> `beatport_scripts_patcher.exe` for you if it sits in the same folder as the
> proxy. Clicking anywhere else on the splash only closes it.
>
> If the patcher next to the proxy is older than the proxy itself, the line and
> the menu item say so and take you to the download instead of running it, since
> an out-of-date patcher may not be able to apply the fix at all. All of this
> stays hidden when there is nothing to do, including when you have no patcher.

> **Faster Beatport tagging** is worth turning on if selecting a track feels
> slow. Beatport sends about 630 KB for a single track, of which the track
> itself is under 2 KB; the rest is the charts and recommendations sections
> that no tag script reads. With this on, the proxy cuts the page down before
> Mp3tag ever sees it, which saves Mp3tag more than a dozen search and replace
> passes over half a megabyte. The difference is largest on older machines and
> does not change a single field you get.
>
> It only ever touches Beatport track pages. Release pages, searches, artwork
> and Traxsource go through untouched. If you have a custom script that reads
> something outside the track's own data, turn it off.

---

## Troubleshooting

**403 Forbidden**: proxy isn't running. Double-click `mp3tag_proxy.exe`.
Also check that no VPN is blocking `127.0.0.1`.

**Beatport is slow on the first request**: that's expected. The proxy is using
Edge, Chrome or Brave in the background to clear Cloudflare's challenge
(~5–10 s, longer on a slow connection or an older machine); it's cached
afterwards, so following requests are fast.

**Connection error / no tray icon**: proxy failed to start. Check `proxy.log`
next to the `.exe`. Port 8787 may be in use by another application (see below to
change it).

**Changing the port**: if 8787 is blocked or taken, create/edit `proxy.cfg` next
to the `.exe` and add a line like `port=9000`. You must also change the port in
your **script URLs** (`http://127.0.0.1:9000/…`) to match. Restart the proxy and
Mp3tag afterwards.

**No results in Mp3tag**: fully exit Mp3tag (File → Exit) and reopen.
Check that the script files are in `%APPDATA%\Mp3tag\data\sources\`.

**Tray icon not visible**: click the `^` arrow on the taskbar near the clock.

---

## Security

Some antivirus engines flag `mp3tag_proxy.exe` on generic ML heuristics (e.g.
Microsoft Defender's `Wacatac.ml`), triggered by behavior the tool genuinely
does: automating a local browser off-screen to pass Cloudflare, and killing
its process afterward. This is a known false-positive pattern on PyInstaller-built
executables; we've verified the published binary's hash matches our build and
that every bundled dependency is an unmodified copy from PyPI. We reported
this to Microsoft and it's been resolved: Defender no longer flags it. If
your own install still shows the old result, it's a cached verdict, run
`MpCmdRun.exe -removedefinitions -dynamicsignatures` followed by
`MpCmdRun.exe -SignatureUpdate` (as administrator) to clear it.

A [software bill of materials](SBOM.json) (CycloneDX format) lists every
dependency bundled into the executable, with versions.

---

## Contact

Questions, issues, or requests: find me on the Mp3tag community forum as **The_Zhu**.

---

## Disclaimer

Mp3tag Proxy is an independent, third-party tool and is not affiliated with or
endorsed by Mp3tag. Mp3tag is a registered trademark and the exclusive property
of its holder, as is its logo.
