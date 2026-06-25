# CLAUDE.md

Guidance for working in this repo. Read before editing `index.html`.

## What this is

A single-file browser game ("The Travels of Marco Polo — A Chengyu Scroll")
that teaches classical Chinese idioms (成語) through a choose-your-own-adventure.
Everything lives in `index.html`: inline CSS and JS, no build step, no
dependencies, no server. To run it, open `index.html` in a browser.

Tracked files: `index.html` (the game), `README.md`, `pamir-draft-v5.txt`
(a prose draft artifact).

## Code structure (all inside the one `<script>` in `index.html`)

- `CHENGYU` — object keyed by pinyin slug (`shibangongbei`, `saiwengshima`,
  `houjibofa`, `jiyuqiucheng`, `yusuzebuda`). Each holds the characters,
  pinyin, gloss, and origin shown on the idiom card and in the glossary.
- `SCENES` — object keyed by scene id: `start`, `acre`, `hormuz`, `pamir`,
  `kashgar`, `khanbaliq`, `ending`. Each scene has `narration` (array of
  strings), `choices` (each with a `next` scene and optionally a `chengyu`
  slug awarded by that choice), and presentation fields.
- `renderEnding`'s `allKeys` array decides which chengyu appear in the
  end-of-game glossary. If you add a scene that awards a new chengyu, add its
  slug here too.

### Current chengyu mapping (the CODE is authoritative)

| Scene  | Awards slug      | Characters   |
|--------|------------------|--------------|
| acre   | `yusuzebuda`     | 欲速則不達   |
| hormuz | `shibangongbei`  | 事半功倍     |
| pamir  | `saiwengshima`   | 塞翁失馬     |
| kashgar| — (flavor only)  | —            |

`houjibofa` and `jiyuqiucheng` are defined in `CHENGYU` but not currently
awarded by any scene.

> Note: `README.md`'s location/chengyu table is **stale** relative to this
> mapping (it predates the Acre/Hormuz re-key). Trust the code, not the README,
> and update the README when convenient.

## Editing gotchas — these have bitten us repeatedly

1. **Escape quotes inside narration strings.** The `chengyu-tip` and
   `inline-pinyin` `<span>`s are embedded in double-quoted JS strings, so every
   inner `"` MUST be escaped as `\"`, e.g.
   `<span class=\"chengyu-tip\" data-key=\"...\" tabindex=\"0\">`. A single
   unescaped quote terminates the string early and throws a `SyntaxError` that
   breaks parsing of the **entire** `SCENES` object — the whole game fails to
   load, not just that scene. Copy the escaping from an existing working scene.

2. **Prefer byte-level Python splices over the Edit tool for anything touching
   Chinese characters.** The file is UTF-8 with **CRLF** line endings and
   contains multi-byte CJK text. The Edit tool has repeatedly mangled Unicode
   escapes on this file. For edits near Chinese characters, read the bytes, do a
   targeted byte replacement in Python, and write back — keep CRLF intact.

3. **Don't trust the Windows console's rendering of CJK.** cp1252 consoles
   render some characters as visually-similar wrong glyphs (e.g. 翁 U+7FC1 shows
   as 翎). Verify by byte sequence (`翁` = `e7 bf 81`), not by eye.

4. **No JS engine is installed here** (no node/deno/bun), so `node --check`
   isn't available. Validate changes by matching the known-good escaping pattern
   of existing scenes, or by opening the file in a browser.
