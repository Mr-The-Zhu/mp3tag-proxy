# Script Modifications

This file describes the exact edits to apply to your Mp3tag tag-source scripts
so they route requests through the local proxy (`mp3tag_proxy.exe`).

The change is the same in all cases: replace the direct URL to the music platform
with `http://127.0.0.1:8787` in specific lines of the script files.

---

## Before you edit

- Edit **only** the `.inc` files and the "Direct" `.src` files listed below.  
  **Do not touch** `.settings` files — editing them causes errors like  
  `ERROR(Settings): expected value, got 'B' (66)`.
- Save as **UTF-8 without BOM**. Use Notepad++ (*Encoding → UTF-8 without BOM*)  
  or VS Code. Classic Notepad may silently add a BOM and corrupt the file.
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

The same pattern applies. In each Traxsource `.inc` file, find `[BasedOn]`
and `[IndexUrl]` and replace `https://www.traxsource.com` with
`http://127.0.0.1:8787`.

```
BEFORE:  [BasedOn]=https://www.traxsource.com
AFTER:   [BasedOn]=http://127.0.0.1:8787

BEFORE:  [IndexUrl]=https://www.traxsource.com/...
AFTER:   [IndexUrl]=http://127.0.0.1:8787/...
```

For any "Direct" `.src` files, change the URL template the same way as
described for Beatport above — replace `https://www.traxsource.com` with
`http://127.0.0.1:8787` in the third `||` field of `[SearchBy]`.

Also change the `# _URL (REQUIRED FOR PREVIEW BUTTON TO WORK)` line
in each Search `.inc` if present.

> **Do not touch** `.settings` files or "Search by…" `.src` files
> that do not contain a direct URL in `[SearchBy]`.

---

## Files that do not need changes

All "Search by…" `.src` files and `.settings` files are used as-is.
These files build only the query string — the URL is taken from `[IndexUrl]`
in the `.inc` file, which is already pointed at the proxy by the edits above.
