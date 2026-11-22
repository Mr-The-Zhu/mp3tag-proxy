# Mp3tag Proxy

A small helper for Mp3tag web sources, for Windows. It runs in the system tray,
sits between Mp3tag and the sites your tag-source scripts read, and does what
Mp3tag cannot do on its own:

- **Beatport, Traxsource and Qobuz**: gets Mp3tag past the sites' bot
  protection (Cloudflare and others), so the **403 Forbidden** errors are gone
- **Beatport and Traxsource**: keeps stevehero's and Jordi's scripts working
  when the sites change their pages
- **Bandcamp**: brings back search by artist and album name, with release dates

Other sites can go through it too: see [For script authors](#for-script-authors).

---

## Requirements

- Windows 10 / 11 with a Chromium browser: Microsoft Edge (preinstalled),
  Google Chrome, or Brave, used to pass Cloudflare checks
- [Mp3tag](https://www.mp3tag.de/), the version your scripts require
  (stevehero's Beatport v6: **v3.22+**, Traxsource: **v3.26+**, Bandcamp: **v3.22+**,
  Qobuz: **v3.22+**)
- Tag-source scripts for the sites you tag from (see below)

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

**Qobuz**: GMK's Qobuz scripts, from his thread:  
[community.mp3tag.de › WS Qobuz](https://community.mp3tag.de/t/60860)

Qobuz turns Mp3tag away on its own, and since version 2026.11.18 these scripts
go through the proxy by themselves.

Place all script files into Mp3tag's sources folder:
```
%APPDATA%\Mp3tag\data\sources\
```

For the Beatport and Traxsource scripts, which it checks and fixes, the proxy
looks there and in the two other places Mp3tag can keep them:

- **Microsoft Store version**: on our test PC, which already had the regular
  Mp3tag, it used the same folder as above. Mp3tag's own documentation gives
  `%LOCALAPPDATA%\Packages\35795FlorianHeidenreich.Mp3tag_rf0p6xgxmspcc\LocalCache\Roaming\Mp3tag\data\sources\`
  instead, so the proxy looks there too. Its **File → Open configuration
  folder** did nothing in that test, so if you need the folder, check both.
- **Portable Mp3tag**: `data\sources` inside its own folder. The proxy finds
  this one when `mp3tag_proxy.exe` is somewhere inside that folder, for
  example in a `data\tools` folder of its own.

If yours are anywhere else, the splash says so once, in grey, when the proxy
starts: **No Beatport or Traxsource scripts**. Click that line, or open
**Settings…** in the tray menu and choose **This folder:**, to tell the proxy
where they are. If you use neither pack, just ignore it.

### 3. Edit the script files

Start `mp3tag_proxy.exe`. It checks both script packs on startup, and if either
needs anything it shows a line on the splash and a **Fix tagging scripts…** item
in the tray menu. Click either one and it applies every edit they need: proxy
routing, full-resolution cover art, Beatport's current JSON fix and Traxsource's
current HTML fix. It keeps a `.bak` of each original first, tells you what it
changed, and puts the full report in the log. If a script is newer than the
proxy knows and an edit does not fit, it says which one instead.

The same check runs at every startup, so if either site changes something later
and an update brings a new fix, the proxy tells you then too.

The Bandcamp and Qobuz scripts need no edits: they already go through the proxy.

Every edit it makes is also written out step by step in
[MODIFICATIONS.md](MODIFICATIONS.md), for anyone who would rather apply them by
hand or read exactly what changed before trusting it.

> **Important:** do not edit `.settings` files.

### 4. Restart Mp3tag

**File → Exit**, then reopen, so Mp3tag reloads the updated scripts.

---

## Daily use

1. Double-click `mp3tag_proxy.exe`. A tray icon appears near the clock. To
   have it there all the time, tick **Start with Windows** in **Settings…**.
2. Use Mp3tag tag sources as usual. Tags are fetched via the proxy.
3. Leave it running, or choose **Quit** in its tray menu when done.

> **Cloudflare checks:** Beatport, and now and then Traxsource, can put a
> Cloudflare check in front of a page, which needs a real browser to pass. When
> one comes, the proxy quietly opens Edge, Chrome or Brave off-screen, passes
> the check on that page, and closes the browser again. That takes about 5–10
> seconds, longer on a slow connection or an older machine; everything after it
> is instant until the check expires. Meanwhile the tray menu shows an orange
> dot and **Passing a Cloudflare check…**. Mp3tag only waits 12 seconds for an
> answer, so when the check takes longer, the proxy asks Mp3tag to hold on
> until the answer is ready.

> **Preview:** the **Preview** button in the Beatport and Traxsource scripts
> opens the page on the site itself, in your browser. The proxy hands Mp3tag
> pages cut down to what the scripts read, which a browser could not show.

> **Beatport releases:** Beatport no longer puts the track list on its release
> pages. The proxy loads each track's own page and puts the list back, so the
> Release scripts work as before, but a release takes a few seconds longer to
> load than a single track.

---

## Tray menu

A left or right click on the tray icon opens it.

- The first line, with the icon, is the proxy's name and version: click it for
  **About**.
- The second line says how things stand, with a coloured dot: green when all is
  well, orange while a Cloudflare check runs, red when a check failed or no
  Edge or Chrome was found. Click it to open the log, which has the detail.
- Grey lines follow: today's requests (in brackets, how many needed a
  Cloudflare check), when the last one came in, and whether the Beatport and
  Traxsource scripts are up to date or need fixing. If it still says **No
  requests yet** while you tag, Mp3tag is not going through the proxy (see
  Troubleshooting).

Below them:

| Item | Action |
|---|---|
| Update available | Shown when there is a newer release: opens its download page |
| Fix tagging scripts… | Shown when the scripts need an edit: applies it (see step 3) |
| Open log | Opens `proxy.log` in your text editor |
| Open proxy folder | Opens the folder with `proxy.cfg` and `proxy.log` in it |
| Settings… | Every setting, see below |
| About | Version and address. Click anywhere on it, press Escape, or use the close mark to dismiss it |
| Quit | Shuts down the proxy |

> On startup the proxy checks GitHub for a newer release (version numbers only,
> nothing personal is sent). If one is available, the splash shows it and the
> menu gets an **Update available** item. The check can be turned off in
> Settings.

---

## Settings

**Settings…** in the tray menu opens a window with every setting, each with a
line saying what it does. They are kept in `proxy.cfg` next to the `.exe`, which
you can also edit by hand while the proxy is not running.

| Setting | What it does | In `proxy.cfg` |
|---|---|---|
| Start with Windows | Starts the proxy quietly in the tray when you sign in. It uses your own startup list, so Windows' **Settings → Apps → Startup** shows it and can turn it off too | (Windows' own) |
| Close after … minutes without requests | Closes the proxy after 15, 30, 60 or 120 minutes without a request. Off by default, and greyed out while Start with Windows is on | `auto_exit=1`, `auto_exit_minutes=30` |
| Faster Beatport tagging | Sends Mp3tag only what the scripts read from Beatport (see below). Off by default | `trim_pages=1` |
| Open Bandcamp search results through the proxy | Only for PCs where Bandcamp turns Mp3tag itself away (see Troubleshooting). Off by default | `bandcamp_pages=1` |
| Beatport and Traxsource scripts | **Find their folder automatically** (the default), or **This folder:** if they are somewhere the proxy cannot guess | `sources=D:\Mp3tag\data\sources` |
| Check for a new version at startup | On by default | `updates_enabled=0` to turn it off |
| Port | 8787 unless that one is taken. Used from the next start, and every script that uses the proxy must use the same port | `port=8787` |

Up to 2.8.4 the proxy closed itself after 30 minutes without requests, and
Mp3tag then found nothing to talk to. From 3.0 it stays until you quit it,
unless you turn that on again.

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

## For script authors

If a site turns Mp3tag away but opens fine in a browser, your script can go
through the proxy: put `http://127.0.0.1:8787/` in front of the site's
address, for instance

```
[IndexUrl]=http://127.0.0.1:8787/https://www.example.com/search?q=%s
```

and do the same in `[AlbumUrl]` if the script opens result pages too. The
proxy fetches the page looking like a real browser and, if Cloudflare asks
for its check, passes it the way it does for Beatport. GMK's Qobuz scripts
work this way.

- **Full addresses:** always give the whole address after the proxy's, with
  `https://`. A bare path (`http://127.0.0.1:8787/search?...`) goes to
  Beatport, because that is how stevehero's scripts use the proxy.
- **Request headers:** some APIs want a key in a header, which Mp3tag scripts
  cannot send. Through the proxy they can: add `proxy_header_<Name>=<value>` to
  the address, for example `&proxy_header_X-API-Key=<your key>`, and the proxy
  takes it out of the address and sends it as the header `<Name>` instead.
  Requests go out looking like a browser, so an API that picks its format from
  `Accept` may need `&proxy_header_Accept=application/json` as well. Header
  values never show up in `proxy.log`.
- **Slow answers:** Mp3tag gives up on a request after 12 seconds. When an
  answer takes longer, the proxy asks Mp3tag to hold on until it is ready, so a
  script needs nothing for that.
- **Looking at a site in a browser:** open it as
  `http://www.example.com.localhost:8787/` rather than with the proxy's address
  in front. The page then keeps its own paths, which suits sites built as web
  apps.

The proxy is made for the sites named at the top. Others may well work too
(Qobuz started that way); if one does not, tell me on the forum.

---

## Troubleshooting

**Mp3tag cannot connect (error 12029)**: the proxy is not running. Start
`mp3tag_proxy.exe`, or tick **Start with Windows** in Settings so it is always
there. Also check that no VPN or firewall is blocking `127.0.0.1`.

**403 Forbidden**: either Mp3tag went to the site directly rather than through
the proxy (see **Still 403 although the proxy is running** below), or the
site's check could not be passed: the tray menu then shows a red dot, and
`proxy.log` the reason.

**Beatport is slow on the first request**: that's expected when a Cloudflare
check comes. The proxy is using Edge, Chrome or Brave in the background to pass
it (about 5–10 s, longer on a slow connection or an older machine); it's kept
afterwards, so following requests are fast.

**No tray icon**: the proxy did not start, or has been closed. Check `proxy.log`
in its folder. Port 8787 may be in use by another program: choose another one
in Settings (see **Changing the port**). If **Close after … minutes** is on in
Settings, the proxy also closes itself after that long without requests.

**The proxy disappears while tagging** (Mp3tag shows error 12030): start it
again and look at the top of the new session in `proxy.log`. If the proxy
crashed, the stack trace from that crash is copied there. If it says the previous
run "was most likely ended from outside", something else closed it, usually
security software reacting to the hidden browser. The `[clearance]` lines just
before show which step it was on. Please include that part of the log when you
report it.

**Changing the port**: if 8787 is blocked or taken, choose another one in
Settings, or add a line like `port=9000` to `proxy.cfg`. It is used from the
next start. Every script that uses the proxy must then say
`http://127.0.0.1:9000/…` instead. The proxy's own fixes point at 8787, so on
any other port it leaves the Beatport and Traxsource scripts alone: make the
edits by hand, as [MODIFICATIONS.md](MODIFICATIONS.md) describes, with your port.

**No results in Mp3tag**: fully exit Mp3tag (File → Exit) and reopen.
Check that the script files are in `%APPDATA%\Mp3tag\data\sources\`.

**Still 403 although the proxy is running**: Mp3tag is going to the site
directly, so the scripts were never pointed at the proxy; the tray menu then
still says **No requests yet** while you tag. For the Beatport and Traxsource
scripts that usually means the proxy did not find them: `proxy.log` lists every
folder it looked in. Open **Settings…**, choose **This folder:** and pick yours,
then click **Fix tagging scripts…**. For other scripts, check that their
addresses start with `http://127.0.0.1:8787/`.

**Bandcamp Search finds nothing**: it needs the proxy running. If it is, look
at `proxy.log`: a `[bandcamp]` line says how many results came back, and says
so if Bandcamp answered with its own bot check instead.

**Bandcamp Search lists results, but opening one gives 403**: on some PCs
Bandcamp turns Mp3tag itself away (seen on an older Windows 10), while the
proxy's own requests get through. Tick **Open Bandcamp search results through
the proxy** in Settings: the results, their thumbnails and the cover then all
come through the proxy. The Bandcamp scripts that look up a URL you give them
still go straight to Bandcamp. To send those through the proxy too, change
`[AlbumUrl]=%s` to `[AlbumUrl]=http://127.0.0.1:8787/%s` in each of them.

**Tray icon not visible**: click the `^` arrow on the taskbar near the clock.

---

## Security

The proxy only listens on your own computer (`127.0.0.1`), and only answers
Mp3tag and pages opened through it: a web page from another site cannot use it
to reach your network or other sites.

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
