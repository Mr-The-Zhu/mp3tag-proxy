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
  **Do not touch** `.settings` files: editing them causes errors like  
  `ERROR(Settings): expected value, got 'B' (66)`.
- Use targeted find/replace, only change the lines listed here.

---

## Beatport: stevehero's v6 scripts

> **Recommended: run `beatport_scripts_patcher.exe`** (bundled with the proxy
> download) instead of doing the edits below by hand. It applies every fix in
> this section automatically, backs up each file before touching it, is safe
> to run more than once, and the proxy itself will tell you (on the splash
> screen and in the tray menu) if it detects your scripts still need it.
>
> The manual steps below are kept for people who can't or don't want to run
> an `.exe` (the patcher is Windows-only; if you're on Mac, or you'd rather
> see and apply every change yourself before trusting it), and as an exact
> record of what changed and why.

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

> **Direct files: change `[BasedOn]` only, do NOT add an `[IndexUrl]` line.**
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

### Optional: full-resolution cover art

Beatport's own JSON sometimes gives a small (e.g. 500×500) `image.uri`, even
though a much larger version exists. There's a second field, `image.dynamic_uri`,
with a `{w}x{h}` size template you can fill in yourself to always get the full
1400×1400 version. This affects the `COVERURL` block in three `.inc` files:
`Release Direct`, `Release Search` and `Artwork Search`.

`Track Direct` and `Track Search` are **not** in that list any more. Beatport's
track page no longer has an `image` object at all, so their cover art is handled
by the 2026-09 schema fix below, which reads `release.image_url` instead. Do not
apply this edit to those two files: it would change the block that the schema fix
expects to find, and that fix would then be reported as not applying.

In each of the three, find:

```
json_select "uri"
SayRest
```

Replace with:

```
json_select "dynamic_uri"
Replace "{w}" "1400"
Replace "{h}" "1400"
SayRest
```

> Order matters: `Replace` must come **before** `SayRest`. It edits the
> currently-selected JSON value, not what's already been output. Putting it
> after `SayRest` leaves the literal `{w}x{h}` in the URL and the cover comes
> back empty.

### Required (2026-09): Beatport changed the track-page JSON schema

Sometime between late August and early September 2026, Beatport rewrote the
data behind every **track** page (`beatport.com/track/...`). The React
query key changed from `track-<id>` to `track-details-<id>`, and almost
every field was renamed or moved. This breaks **`Track Direct.inc`** and
**`Track Search.inc`** (both fetch a track page to fill in the tag fields),
with symptoms of empty/wrong `TITLE`, and `ARTIST` mixing in remixer names. It does
**not** affect `Release Direct.inc` / `Release Search.inc` / `Artwork
Search.inc`: release pages (`beatport.com/release/...`) were not changed.

What moved, for reference: `name`→`track_name`; `id`→`track_id`; `length`
(an `"m:ss"` string)→`track_length_ms` (milliseconds); `new_release_date`
(track-level)→`release.release_date`, **and it is now a full timestamp**
(`"2019-02-15T00:00:00"`) where the old field was a plain date, so it has to
be trimmed before any date formatting, exactly as the search-results parser
in `Track Search.inc` already does with
`RegexpReplace "T\d+:\d+:\d+" ""`; `catalog_number` (track-level)→
`release.catalog_number`; `release.image.uri`/`.dynamic_uri`→a single
`release.image_url` (still a `{w}x{h}` template); `release.label`→`label`
(now a sibling of `release`, not nested under it); `key` (an object with
`camelot_number`/`camelot_letter`/`name`)→a plain string (e.g. `"F Minor"`)
plus a new ready-made `key_camelot` string (e.g. `"4A"`); the separate
`remixers` array is gone, remixers are now just entries in `artists` with
`"type":"Remixer"` (main artists have `"type":"Artist"`); each artist's
`url` field is gone, replaced with `slug`; `sub_genre` moved from a
top-level field into `genre.sub_genre` and is now always present (with a
`null` name) instead of sometimes being absent entirely; `exclusive` moved
into `price.category`, an array of plain strings that contains `"exclusive"`
when the track is one; and `desc` (the track description) is gone from this
endpoint entirely, though it is still on the release page.

**Fix:** in both `Track Direct.inc` and `Track Search.inc`, replace the
entire block starting on the line *after* the comment `# Fix the artist URL
from the API one to the normal beatport URL` (keep that comment itself, which
is what the patcher does) and ending at the last `Endif` of the `YEAR`
section (i.e. everything from just after the "USER OPTIONS" block down to
the "TAG CUSTOMIZATION" divider) with:

```
# --- 2026-09 fix: Beatport changed the track-page JSON schema ---
# The track detail page's React Query key changed from "track-<id>" to
# "track-details-<id>" and almost every field was renamed/restructured.
# Artists and remixers used to be two separate arrays; now they're one
# "artists" array with a "type" field ("Artist"/"Remixer"). There is no
# ready-made artist page "url" any more either. These two regexes give
# artists a distinct field name per type (same trick the TRACK Search
# index parser already uses) and rebuild a real artist URL from the new
# "id"+"slug" fields.
RegexpReplace "(\"id\":\s*)(\d+)(\s*,\s*\"name\"\s*:\s*\"[^\"]*\"\s*,\s*\"type\"\s*:\s*\"(?:Artist|Remixer)\"\s*,\s*\"slug\"\s*:\s*\")([^\"]+)(\")" "$1$2$3$4$5,\"artist_url\":\"https://www.beatport.com/artist/$4/$2\""
RegexpReplace "(\"id\":\s*\d+\s*,\s*\")name(\"\s*:\s*\"[^\"]*\"\s*,\s*\"type\"\s*:\s*\"Artist\")" "$1actual_artist_name$2"
RegexpReplace "(\"id\":\s*\d+\s*,\s*\")name(\"\s*:\s*\"[^\"]*\"\s*,\s*\"type\"\s*:\s*\"Remixer\")" "$1remixer_artist_name$2"

# "exclusive" is not gone, it moved: a track is exclusive when the string
# "exclusive" appears in "price"."category", which is an array of plain
# strings. There is no way to test array membership in the JSON engine, so
# these two build an "is_exclusive" flag next to it in the same 1/0 shape
# the old field had. The first marks the ones that are exclusive; the
# second then marks everything still unmarked, which is why the order of
# these two lines matters. Thanks to stevehero for pointing this out.
RegexpReplace "(\"price\":\{\"category\":\[[^\]]*\"exclusive\"[^\]]*\])" "$1,\"is_exclusive\":\"1\""
RegexpReplace "(\"price\":\{\"category\":\[[^\]]*\]),\"currency\"" "$1,\"is_exclusive\":\"0\",\"currency\""

RegexpReplace "(?i)\bep\b" "EP"                               # Fix Ep to EP
RegexpReplace "\s+Remix\)\s+\(Original\s+Mix\)" " Remix)"     # Fix ' Remix (Original Mix)'
Replace "\u2018" "'"                                          # Fix left quote to apostrophe
Replace "\u2019" "'"                                          # Fix right quote to apostrophe
Replace "Deadmau5" "deadmau5"

# Turn on JSON for the remainder of the script
json "ON" "current"

# Set it up to go into the data object
json_select_object "props"
json_select_object "pageProps"
json_select_object "dehydratedState"
json_select_array "queries" 1 # Select "queries" 1 array (Album info)
json_select_object "state"
json_select_object "data"

OutputTo "_TIME_CHECK"  # Shows track length when only a single track is available, , convenient to check time comparison
json_select "track_length_ms" # was "length" (an "m:ss" string); now milliseconds under a new name
SayDuration "ms"

OutputTo "ALBUM"
json_select_object "release"
json_select "name"
SayRest
json_unselect_object

OutputTo "ARTIST"
json_select_many "artists" "actual_artist_name" ", " " & " # was "name", filtered to type=Artist now (see regexes above), otherwise remixers leak in
SayRest

OutputTo "BPM"
json_select "bpm"
SayRest

OutputTo "COVERURL"
json_select_object "release"
json_select "image_url" # was release.image.uri; the "image" sub-object is gone, this is now a {w}x{h}-templated URL directly on release
Replace "{w}" "1400"
Replace "{h}" "1400"
SayRest
json_unselect_object

OutputTo "DATE"
IfVar "SettingDateFormat" "MMDD"
json_select_object "release"
json_select "release_date" # was top-level "new_release_date", moved under release
RegexpReplace "T\d+:\d+:\d+" ""  # release_date is a full timestamp now, same trim the search parser uses
RegexpReplace "\d\d\d\d-(\d\d)-(\d\d)" "$1$2"     # DATE in MMDD format
SayRest
json_unselect_object
Endif
IfVar "SettingDateFormat" "DDMM"
json_select_object "release"
json_select "release_date"
RegexpReplace "T\d+:\d+:\d+" ""  # release_date is a full timestamp now, same trim the search parser uses
RegexpReplace "\d\d\d\d-(\d\d)-(\d\d)" "$2$1"     # DATE in DDMM format
SayRest
json_unselect_object
Endif
IfVar "SettingDateFormat" "YYYY-MM-DD"
json_select_object "release"
json_select "release_date"
RegexpReplace "T\d+:\d+:\d+" ""  # release_date is a full timestamp now, same trim the search parser uses
SayRest
json_unselect_object
Endif

OutputTo "BEATPORT_ARTIST_URL"
json_select_many "artists" "artist_url" "\\u005c\\u005c" # built above from id+slug; the API no longer gives a ready-made "url"

SayRest

OutputTo "BEATPORT_LABEL_URL"
Say "https://www.beatport.com/label/"
json_select_object "label" # was release.label; "label" moved out to be a sibling of "release"
json_select "slug"
SayRest
Say "/"
json_select "id"
SayRest
json_unselect_object

OutputTo "BEATPORT_RELEASE_ID"
json_select_object "release"
json_select "id"
SayRest
json_unselect_object

OutputTo "BEATPORT_TRACK_ID"
json_select "track_id" # was "id"
SayRest

OutputTo "BEATPORT_TRACK_URL"
Say "https://www.beatport.com/track/x/" # was a real "slug" field. Beatport dropped it from this endpoint; confirmed live that the slug text is ignored and the page loads from the id alone, so a placeholder is harmless
json_select "track_id" # was "id"
SayRest

OutputTo "GENRE"
json_select_object "genre"
json_select "name"
IfVar "SettingGenreRename" "true"
RegexpReplace "\s*(Dee|Ele|Min|Pro|Tec)(.+?)\s+(House)\s*" "$3 $1$2"  # 'Progressive House' => 'House Progressive', 'Electro House' => 'House Electro', etc.
Endif
SayRest
# "sub_genre" moved inside "genre" (used to be a top-level sibling). Beatport now always sends a
# sub_genre object even when there isn't one (its own "name" comes back null instead). Unlike
# before, json_select_object "sub_genre" always succeeds and must always be unselected: check the
# nested "name" value itself for emptiness, not whether entering the object left an empty selection.
json_select_object "sub_genre"
json_select "name"
IfNot ""
Say " / "
SayRest
Endif
json_unselect_object
json_unselect_object
Say "|"

OutputTo "INITIALKEY"
# "key" used to be an object ({letter, chord_type, camelot_number, camelot_letter, ...}). Beatport now
# sends it as a plain string ("F Minor") plus a ready-made combined "key_camelot" string ("4A"), so
# there's no object to select into any more.
IfVar "SettingInitialkeyFormat" "Camelot"
json_select "key_camelot"
IfVar "SettingInitialkeyAddPadding" "true"
RegexpReplace "^\s*(\d)([AB])\s*$" "0$1$2" # 8A => 08A, 7B => 07B, etc.
Endif
SayRest
Endif
IfVar "SettingInitialkeyFormat" "Musical"
json_select "key"
Replace "b" "♭"
SayRest
Endif
Say "|"

OutputTo "ISRC" # International Standard Recording Code
json_select "isrc"
SayRest

OutputTo "MIXARTIST"
json_select_many "artists" "remixer_artist_name" ", " " & " # was a separate top-level "remixers" array, merged into "artists" now, filtered by type (see regexes above)
SayRest

OutputTo "PUBLISHER"
json_select_object "label" # was release.label
json_select "name"
SayRest
json_unselect_object

OutputTo "TITLE"
json_select "track_name" # was "name"
SayRest
json_select "mix_name"
RegexpReplace "\(+(.+?)\)+" "$1" # Removes any brackets for the mix_name property as the code below will add them. NOTE: Some mix_names don't have any, so this is needed.
IfNot ""
Say " ("
SayRest
Say ")"
Endif

OutputTo "UNSYNCEDLYRICS"
Say "Release type:\\u0009\\u0009"
Say "Beatport Single Track"
Say "\\u000d\\u000a----------------------------------------------\\u000d\\u000a"
Say "Exclusive to beatport:\\u0009"
# was a plain "exclusive" 0/1 field; it now lives as a string inside
# "price"."category", and the regexes at the top turn that back into 1/0.
json_select_object "price"
json_select "is_exclusive"
Replace "0" "❎"
Replace "1" "✅"
SayRest
json_unselect_object
Say "\\u000d\\u000a----------------------------------------------\\u000d\\u000a"
# "desc" (the track description) really is gone from this endpoint. It is still on the
# release page, but a track script never fetches one, so that line is dropped rather
# than guessed at.
Say "Tagged by:\\u0009\\u0009Mp3Tag w/ beatport.com scripts [v6.007 by stevehero™] (◣_◢) (http://bit.ly/2EmyidV)"

OutputTo "WWW"
Say "https://www.beatport.com/release/"
json_select_object "release"
json_select "slug"
sayrest
Say "/"
json_select "id"
sayrest
json_unselect_object
sayrest

OutputTo "YEAR"
IfVar "SettingYearFormat" "DD-MM-YYYY"
json_select_object "release"
json_select "release_date" # was top-level "new_release_date"
RegexpReplace "T\d+:\d+:\d+" ""  # release_date is a full timestamp now, same trim the search parser uses
RegexpReplace "(\d\d\d\d)-(\d\d)-(\d\d)" "$3-$2-$1"   # YEAR in DD-MM-YYYY format
SayRest
json_unselect_object
Endif
IfVar "SettingYearFormat" "YYYY"
json_select_object "release"
json_select "release_date"
RegexpReplace "T\d+:\d+:\d+" ""  # release_date is a full timestamp now, same trim the search parser uses
RegexpReplace "(\d\d\d\d)-\d\d-\d\d" "$1"             # YEAR in YYYY format
SayRest
json_unselect_object
Endif
IfVar "SettingYearFormat" "YYYY-MM-DD"
json_select_object "release"
json_select "release_date"
RegexpReplace "T\d+:\d+:\d+" ""  # release_date is a full timestamp now, same trim the search parser uses
SayRest
json_unselect_object
Endif
```

Verified live against three real tracks (a plain single-artist track, a
track with a remixer, and a third unrelated track): `TITLE`, `ARTIST`,
`MIXARTIST`, `COVERURL`, `INITIALKEY` (both Musical and Camelot), `GENRE`,
`ISRC`, `BPM`, `YEAR`/`DATE`, `PUBLISHER` and all three `BEATPORT_*_URL`
fields all came back correct in every case.

---

## Traxsource: Beatport · **Traxsource** · SoundCloud scripts (by Jordi & Claude)

Traxsource needs a different URL form than Beatport. Beatport is the proxy's
default target, so a bare `http://127.0.0.1:8787/...` is forwarded straight to
Beatport. Traxsource is **not** the default, so its request URLs must embed the
full original address after the proxy:

```
http://127.0.0.1:8787/https://www.traxsource.com/...
```

### 1. `[BasedOn]`: bare proxy

```
BEFORE:  [BasedOn]=https://www.traxsource.com
AFTER:   [BasedOn]=http://127.0.0.1:8787
```

### 2. `[IndexUrl]` and `[AlbumUrl]`: embedded form

Keep the rest of the path; just insert the proxy in front of the original URL.

```
BEFORE:  [IndexUrl]=https://www.traxsource.com/search/tracks?term=%s
AFTER:   [IndexUrl]=http://127.0.0.1:8787/https://www.traxsource.com/search/tracks?term=%s

BEFORE:  [AlbumUrl]=https://www.traxsource.com/track/%s
AFTER:   [AlbumUrl]=http://127.0.0.1:8787/https://www.traxsource.com/track/%s
```

### 3. Search result `_URL`: embedded form

In each Search `.inc`, inside the output loop, the line that builds the result
URL prefixes the host with `say`. Embed the proxy there too:

```
BEFORE:  say "https://www.traxsource.com"
AFTER:   say "http://127.0.0.1:8787/https://www.traxsource.com"
```

### 4. Large covers: use `og:image`, not the thumbnail

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

### 5. Track Search: fix broken title/URL match (Traxsource site change)

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
These files build only the query string. The URL is taken from `[IndexUrl]`
in the `.inc` file, which is already pointed at the proxy by the edits above.
