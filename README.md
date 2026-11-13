# Mp3tag Proxy

Fixes **403 Forbidden** errors when using Mp3tag tag-source scripts with
Beatport and Traxsource, caused by Cloudflare bot protection, and keeps those
scripts working when the sites change their pages. It also brings back
Bandcamp search by artist and album name, with release dates.

The proxy runs locally on your machine and sits between Mp3tag and the music
platforms. It appears as a small icon in the system tray.

---

## Requirements

- Windows 10 / 11 with a Chromium browser: Microsoft Edge (preinstalled),
  Google Chrome, or Brave, used to clear Beatport's Cloudflare challenge
- [Mp3tag](https://www.mp3tag.de/), the version your scripts require
  (stevehero's Beatport v6: **v3.22+**, Traxsource: **v3.26+**, Bandcamp: **v3.22+**)
- Tag-source scripts for Beatport, Traxsource or Bandcamp (see below)

---

## Installation

### 1. Download

Get `mp3tag_proxy.exe` from the [Releases](../../releases/latest) page and put
it in any folder (e.g. `C:\Tools\mp3tag_proxy\`). That one file is all you need.

`mp3tag_scripts_patcher.exe` is on the same page and is **optional**. The
proxy carries the same fixes inside itself and applies them from its own tray
menu, so most people never need it. Download it if you want a per-file report
in a console. It finds the scripts the same way the proxy does (see step 2),
and it can also work on any folder passed on the command line or typed in
when it asks.

It was called `beatport_scripts_patcher.exe` up to v2.6.1, before it learned
the Traxsource scripts as well. If you have that one, or any other older copy,
sitting next to the proxy, delete it or leave it as you prefer: the proxy
ignores it either way.

### 2. Install your tag-source scripts

**Beatport**: stevehero's Beatport v6 scripts, from his own thread:  
[community.mp3tag.de › WS Beatport by stevehero](https://community.mp3tag.de/t/ws-beatport-com-by-stevehero-release-single-track-artwork-tagging/12568)

The fixes are written against **v6.007**, the current release in that thread.
It is the only set of Beatport scripts needed here.

**Traxsource**: Jordi & Claude's scripts:  
[community.mp3tag.de › WS Beatport · Traxsource · SoundCloud](https://community.mp3tag.de/t/ws-beatport-traxsource-soundcloud-updated-fixed-scripts-2026-by-jordi-claude/71123)

That thread is titled Beatport · Traxsource · SoundCloud, but only its
Traxsource scripts are used here. Nothing in this project needs a Beatport
script from it, so if the Beatport files in that thread are unavailable, it
does not affect the proxy or the patcher: use stevehero's above.

**Bandcamp**: the Bandcamp source, **0.5.0** or later, from its thread:  
[community.mp3tag.de › WS Bandcamp](https://community.mp3tag.de/t/17838)

Its **Bandcamp Search** finds albums and tracks by name and shows their
release dates. Bandcamp's own search page needs JavaScript, which Mp3tag cannot
run, so this one works only while the proxy is running. Without the proxy,
0.5.0 works just like 0.4.9, minus Bandcamp Search.

Place all script files into Mp3tag's sources folder:
```
%APPDATA%\Mp3tag\data\sources\
```

The proxy finds them there, and also in the two other places Mp3tag can keep
them:

- **Microsoft Store version**: on our test PC, which already had the regular
  Mp3tag, it used the same folder as above. Mp3tag's own documentation gives
  `%LOCALAPPDATA%\Packages\35795FlorianHeidenreich.Mp3tag_rf0p6xgxmspcc\LocalCache\Roaming\Mp3tag\data\sources\`
  instead, so the proxy looks there too. Its **File → Open configuration
  folder** did nothing in that test, so if you need the folder, check both.
- **Portable Mp3tag**: `data\sources` inside its own folder. The proxy finds
  this one when `mp3tag_proxy.exe` is somewhere inside that folder, for
  example in a `data\tools` folder of its own.

If yours are anywhere else, the splash says so in grey when the proxy starts:
**Tagging scripts not found**. Click that line, or use the tray menu,
**Scripts folder**, **Choose a folder…**, and tell the proxy once where they
are. It is saved in `proxy.cfg` as a line like
`sources=D:\Mp3tag\data\sources`, which you can also add by hand.

### 3. Edit the script files

Start `mp3tag_proxy.exe`. It checks both script packs on startup, and if either
needs anything it shows a line on the splash and a **Fix tagging scripts…** item
in the tray menu. Click either one and it applies every edit they need: proxy
routing, full-resolution cover art, Beatport's current JSON fix and Traxsource's
current HTML fix. It keeps a `.bak` of each original first, tells you what it
changed, and puts the full report in the log.

The same check runs at every startup, so if either site changes something later
and an update brings a new fix, the proxy tells you then too.

The Bandcamp scripts need no edits: their search already goes through the proxy.

Every edit it makes is also written out step by step in
[MODIFICATIONS.md](MODIFICATIONS.md), for anyone who would rather apply them by
hand or read exactly what changed before trusting it.

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
> brief browser process closes itself. Traxsource can ask for the same check now
> and then, and the proxy handles it the same way. Mp3tag only waits 12 seconds
> for an answer, so on a slower PC, where the check takes longer, the proxy asks
> Mp3tag to hold on until the answer is ready.

> **Beatport releases:** Beatport no longer puts the track list on its release
> pages. The proxy loads each track's own page and puts the list back, so the
> Release scripts work as before, but a release takes a few seconds longer to
> load than a single track.

> **Other sites:** the proxy is made for the Beatport, Traxsource and Bandcamp
> scripts, but it can fetch other sites too, for instance while you test a tag
> source of your own: `http://127.0.0.1:8787/https://www.example.com/...`. To
> look at a site in a browser, open it as `http://www.example.com.localhost:8787/`
> instead, which also suits sites built as web apps. Nothing beyond Beatport,
> Traxsource and Bandcamp is tested or supported.

---

## Tray menu

At the top, four grey lines show how things stand:

- the version and the address the proxy runs on, or **Solving the Cloudflare
  check…** while one runs;
- when the last request came in. If this still says **No requests yet** while
  you tag, Mp3tag is not going through the proxy, which usually means the
  scripts were never pointed at it (see Troubleshooting);
- today's requests and Cloudflare checks;
- whether your tagging scripts are up to date, need fixing, or were not found.

Below them:

| Item | Action |
|---|---|
| Open log | Opens `proxy.log` in your text editor |
| Auto-exit when idle (30 min) | Toggle idle auto-exit on/off |
| Faster Beatport tagging | Send Mp3tag less data from Beatport. Off by default |
| Check for updates | Toggle the startup version check |
| Scripts folder | **Find automatically** (the default), or **Choose a folder…** if your scripts are somewhere the proxy cannot guess |
| About | Shows version info. Click anywhere on it, press Escape, or use the close mark to dismiss it |
| Quit | Shuts down the proxy |

> On startup the proxy checks GitHub for a newer release (version numbers only,
> nothing personal is sent). If one is available, the About splash shows it and the
> tray gets an **Update available** item. Turn the check off with the
> **Check for updates** toggle.

> The proxy also checks stevehero's Beatport scripts and Jordi's Traxsource
> scripts (whichever you have) on startup. If they need an edit, a red line
> appears on the splash naming the pack, and the tray gets a **Fix tagging
> scripts…** item. Clicking that line, or that item, applies the fixes there and
> then, keeping a `.bak` of each original, and shows what it changed. The full
> per-file report goes to the log. Clicking anywhere else on the splash only
> closes it.
>
> All of this stays hidden when there is nothing to do. If the proxy finds
> neither set of scripts anywhere, the splash shows a grey **Tagging scripts not
> found** line instead: click it to choose their folder, or ignore it if you
> use scripts of your own.

> **Faster Beatport tagging** is worth turning on if Beatport tagging feels
> slow. It cuts down what Mp3tag has to work through, in the places where
> Beatport sends far more than any tag script uses. It changes no field you get,
> and the difference is largest on older machines.
>
> *The track page.* Beatport sends about 630 KB for a single track, of which the
> track itself is under 2 KB; the rest is the charts and recommendations
> sections that no tag script reads. The proxy cuts the page down before Mp3tag
> ever sees it, saving Mp3tag more than a dozen search and replace passes over
> half a megabyte.
>
> *The release page.* About 350 KB, of which the release and its tracks are a
> few KB. It is cut down the same way.
>
> *The search results list.* A Beatport search returns around 150 results, and
> Beatport now points every one of them at a full 1400x1400 cover, roughly
> 169 KB each. Mp3tag downloads and decodes all of them just to draw the small
> thumbnails in the chooser: about 25 MB of images for a single search, and
> enough work to make an older machine struggle. The proxy asks Beatport's own
> resizer for list-sized covers instead, around 65 times smaller. **The artwork
> written to your files is not affected**, because that comes from the release
> or track itself, which is always kept whole.
>
> Everything on Traxsource passes through untouched either way. If you have a
> custom script that reads something on a track or release page beyond the
> track or release itself, turn it off.

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

**The proxy disappears while tagging** (Mp3tag shows error 12030): start it
again and look at the top of the new session in `proxy.log`. If the proxy
crashed, the stack trace from that crash is copied there. If it says the previous
run "was most likely ended from outside", something else closed it, usually
security software reacting to the hidden browser. The `[clearance]` lines just
before show which step it was on. Please include that part of the log when you
report it.

**Changing the port**: if 8787 is blocked or taken, create/edit `proxy.cfg` next
to the `.exe` and add a line like `port=9000`. You must also change the port in
your **script URLs** (`http://127.0.0.1:9000/…`) to match. Restart the proxy and
Mp3tag afterwards.

**No results in Mp3tag**: fully exit Mp3tag (File → Exit) and reopen.
Check that the script files are in `%APPDATA%\Mp3tag\data\sources\`.

**Still 403 although the proxy is running**: Mp3tag is going to the site
directly, so the scripts were never pointed at the proxy; the tray menu then
still says **No requests yet** while you tag. Usually that means
the proxy did not find them. The splash then shows a grey **Tagging scripts
not found** line, and `proxy.log` lists every folder it looked in. Click that
line, or use the tray menu, **Scripts folder**, to choose yours, then click
**Fix tagging scripts…**.

**Bandcamp Search finds nothing**: it needs the proxy running. If it is, look
at `proxy.log`: a `[bandcamp]` line says how many results came back, and says
so if Bandcamp answered with its own bot check instead.

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
