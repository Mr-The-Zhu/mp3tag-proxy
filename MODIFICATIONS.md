# Script Modifications

This file describes the exact edits to apply to your Mp3tag tag-source scripts
so they route requests through the local proxy (`mp3tag_proxy.exe`).

The change is the same in all cases: replace the direct URL to the music platform
with `http://127.0.0.1:8787` in specific lines of the script files.

> If you changed the proxy's port (via `port=` in `proxy.cfg`), use that port in the
> URLs below instead of `8787`.

---

## Before you edit

- Edit **only** the `.inc` files and the "Direct" `.src` files listed below.  
  **Do not touch** `.settings` files — editing them causes errors like  
  `ERROR(Settings): expected value, got 'B' (66)`.
- Use targeted find/replace — only change the lines listed here.

---

## Beatport — stevehero's v6 scripts

### `.inc` files (5 files)

#### `Beatport by &stevehero v6_Track Direct.inc`
```
BEFORE:  [BasedOn]=https://www.beatport.com/
AFTER:   [BasedOn]=http://127.0.0.1:8787/
```

#### `Beatport by &stevehero v6_Release Direct.inc`
```
BEFORE:  [BasedOn]=https://www.beatport.com
AFTER:   [BasedOn]=http://127.0.0.1:8787
```

> **Direct files: change `[BasedOn]` only — do NOT add an `[IndexUrl]` line.**
> `[IndexUrl]` belongs solely to the *Search* `.inc` files (below). The Direct
> sources have no index parser, so adding an `[IndexUrl]` makes Mp3tag take the
> wrong path and return *"no entries matching your search criteria"* (or a
> connection error). If you see that, check that your Direct `.inc` has no
> `[IndexUrl]` line.

#### `Beatport by &stevehero v6_Track Search.inc`
```
BEFORE:  [BasedOn]=https://www.beatport.com
         [IndexUrl]=https://www.beatport.com/search/tracks?q=
AFTER:   [BasedOn]=http://127.0.0.1:8787
         [IndexUrl]=http://127.0.0.1:8787/search/tracks?q=
```

#### `Beatport by &stevehero v6_Release Search.inc`
```
BEFORE:  [BasedOn]=https://www.beatport.com
         [IndexUrl]=https://www.beatport.com/search/releases?q=
AFTER:   [BasedOn]=http://127.0.0.1:8787
         [IndexUrl]=http://127.0.0.1:8787/search/releases?q=
```

#### `Beatport by &stevehero v6_Artwork Search.inc`
```
BEFORE:  [BasedOn]=https://www.beatport.com
         [IndexUrl]=https://www.beatport.com/search/releases?q=
AFTER:   [BasedOn]=http://127.0.0.1:8787
         [IndexUrl]=http://127.0.0.1:8787/search/releases?q=
```

Also in each Search `.inc`, find the single line marked
`# _URL (REQUIRED FOR PREVIEW BUTTON TO WORK)` inside `[ParserScriptIndex]`
and change the URL there too:

```
BEFORE:  Say "https://www.beatport.com/release/releases/"
AFTER:   Say "http://127.0.0.1:8787/release/releases/"

BEFORE:  Say "https://www.beatport.com/track/tracks/"
AFTER:   Say "http://127.0.0.1:8787/track/tracks/"
```

### "Direct" `.src` files (4 files)

Change the URL template (third `||`-separated field on the `[SearchBy]` line):

#### `...#TRACK Direct by BEATPORT_TRACK_&URL.src`
```
BEFORE:  ...)||%s
AFTER:   ...)||http://127.0.0.1:8787/%s
```

#### `...#RELEASE Direct by &Www(URL).src`
```
BEFORE:  ...)||%s
AFTER:   ...)||http://127.0.0.1:8787/%s
```

#### `...#RELEASE Direct by &BEATPORT RELEASE ID.src`
```
BEFORE:  ...||https://www.beatport.com/release/releases/%s
AFTER:   ...||http://127.0.0.1:8787/release/releases/%s
```

#### `...#TRACK Direct by BEATPORT_TRACK_I&D.src`
```
BEFORE:  ...||https://www.beatport.com/track/tracks/%s
AFTER:   ...||http://127.0.0.1:8787/track/tracks/%s
```

---

## Traxsource — Beatport · **Traxsource** · SoundCloud scripts (by Jordi & Claude)

Traxsource needs a different URL form than Beatport. Beatport is the proxy's
default target, so a bare `http://127.0.0.1:8787/...` is forwarded straight to
Beatport. Traxsource is **not** the default, so its request URLs must embed the
full original address after the proxy:

```
http://127.0.0.1:8787/https://www.traxsource.com/...
```

### 1. `[BasedOn]` — bare proxy

```
BEFORE:  [BasedOn]=https://www.traxsource.com
AFTER:   [BasedOn]=http://127.0.0.1:8787
```

### 2. `[IndexUrl]` and `[AlbumUrl]` — embedded form

Keep the rest of the path; just insert the proxy in front of the original URL.

```
BEFORE:  [IndexUrl]=https://www.traxsource.com/search/tracks?term=%s
AFTER:   [IndexUrl]=http://127.0.0.1:8787/https://www.traxsource.com/search/tracks?term=%s

BEFORE:  [AlbumUrl]=https://www.traxsource.com/track/%s
AFTER:   [AlbumUrl]=http://127.0.0.1:8787/https://www.traxsource.com/track/%s
```

### 3. Search result `_URL` — embedded form

In each Search `.inc`, inside the output loop, the line that builds the result
URL prefixes the host with `say`. Embed the proxy there too:

```
BEFORE:  say "https://www.traxsource.com"
AFTER:   say "http://127.0.0.1:8787/https://www.traxsource.com"
```

### 4. Large covers — use `og:image`, not the thumbnail

The TRACK scripts grab the cover from the `tr-image` block, which is only a
**175×175** thumbnail. Switch them to the `og:image` meta tag, which is the
full **600×600** cover (the RELEASE scripts already use `og:image`).

In `TRACK Direct by ID.inc` and the `[ParserScriptAlbum]` section of
`TRACK Search.inc`:

```
BEFORE:  regexpreplace "<div class=\"tr-image\"><a href=\"[^\"]+\"><img src=\"([^\"]+)\"" "<CoverURL=$1>"
AFTER:   regexpreplace "<meta property=\"og:image\" content=\"([^\"]+)\">" "<CoverURL=$1>"
```

> The "Direct" `.src` files for Traxsource take an ID only (no embedded URL),
> so they need **no** changes. **Do not touch** `.settings` files.

### 5. Track Search — fix broken title/URL match (Traxsource site change)

Traxsource added a small icon inside the track title link on search result pages,
which breaks the regex that reads the track's title and URL. When this happens,
the search silently falls back to fetching the Traxsource homepage instead of the
actual track, giving an empty title and a random mix of unrelated artist names.

In `TRACK Search.inc`, find:

```
BEFORE:  regexpreplace "<!--DIV title--><a href=\"([^\"]+)\">([^<]+)</a>" "<TrackURL=$1><TrackName=$2>"
AFTER:   regexpreplace "<!--DIV title--><a href=\"([^\"]+)\">(?:<img[^>]*>)?([^<]+)</a>" "<TrackURL=$1><TrackName=$2>"
```

---

## Files that do not need changes

All "Search by…" `.src` files and `.settings` files are used as-is.
These files build only the query string — the URL is taken from `[IndexUrl]`
in the `.inc` file, which is already pointed at the proxy by the edits above.
