# Full-Story Revision — Design Spec

**Date:** 2026-06-25
**Branch:** `polo-revision/full-story`
**File touched:** `index.html` (single-file game; inline CSS/JS)

## Goal

Bring the four unrevised scenes (Venice, Kashgar, Khanbaliq, ending) up to the
literary quality of the already-revised Pamir v5 scene, wire every scene to a
chengyu so the whole game teaches one idiom per stop, and replace the current
"branch-and-live-with-it" choice model with a guided model where a wrong choice
is a setback that loops the player back to choose again.

## Decisions (from brainstorming)

1. **Scope:** prose elevation **plus** chengyu wiring (not a deeper structural
   redesign).
2. **Idioms:** six total, one per scene. The two previously-orphaned idioms
   (`厚積薄發`, `急於求成`) get used; Venice's inline proverb is promoted to a
   full card.
3. **Continuity:** thread Niccolò/Maffeo (and the desert guide) through all
   scenes; write cross-scene callbacks directly into prose.
4. **Pacing:** uniform Pamir-weight — every revised scene ~5 dense paragraphs.
5. **Wrong choices loop back:** a wrong pick triggers a Mario-style setback
   that redirects to the same scene to choose again.
6. **Ending:** a single ending for everyone (no score-based variants).
7. **Execution:** Approach A — wire the mechanical skeleton first and verify it
   loads, then write prose scene-by-scene on top of a working game.

## Idiom architecture

Six idioms, in journey order:

| Scene | Slug | Idiom | Lesson | Status |
|---|---|---|---|---|
| Venice (prologue) | `bujikuibu` | 不積跬步 | A thousand li begins with single steps | new card, always earned |
| Acre | `yusuzebuda` | 欲速則不達 | Haste prevents arrival | unchanged idiom |
| Hormuz | `shibangongbei` | 事半功倍 | Knowledge halves the labor | unchanged idiom |
| Pamir | `saiwengshima` | 塞翁失馬 | A loss may be a blessing | unchanged idiom |
| Kashgar | `jiyuqiucheng` | 急於求成 | Impatience sabotages success | newly wired |
| Khanbaliq | `houjibofa` | 厚積薄發 | Accumulate deeply, release sparingly | newly wired |

- **New `CHENGYU` entry `bujikuibu`:** `chars: "不積跬步"` (4 chars, to match the
  card's hardcoded `四字成語` label), `pinyin: "bù jī kuǐ bù, wú yǐ zhì qiān lǐ"`,
  with `literal`/`meaning` carrying the full "...無以至千里" sense. Origin: the
  *Xunzi* (荀子・勸學).
- **Glossary `allKeys`** expands 4 → 6, in journey order:
  `["bujikuibu","yusuzebuda","shibangongbei","saiwengshima","jiyuqiucheng","houjibofa"]`.

## Mechanics changes (engine)

All changes are backward-compatible with the three already-revised scenes'
data shape, except the removal of scoring.

### Choice model

Each decision scene's choices are one of two kinds:

```js
// right choice — advances, teaches the idiom
{ label, next, chengyu }

// wrong choice — setback, loops back to the same scene
{ label, setback: ["…consequence paragraph…", "…"] }
```

The choice click handler branches on which field is present:
- `chengyu` present → show the idiom card inline, then advance to `next`
  (existing `showChengyuThenAdvance` flow, unchanged).
- `setback` present → render a setback interstitial (the consequence
  paragraphs + a **"Try again"** button) that re-renders the *current* scene.

`renderScene` must know the current scene id so the retry can re-render it.
The current scene id is available via `goTo`/`state.history`; pass it through
to the setback handler.

### Scoring removed

Remove `state.score`, the `+1 / −1` lines in the choice handler, and the
variant-selection block in `renderEnding`. Nothing else reads `score`.

### Narration stays static

Narration remains a plain array of strings. Because the path is deterministic
(every finisher made every right choice), cross-scene callbacks are written
directly into prose — no conditional/function narration, no decision tracking.

### Prologue awards its card

Venice's single "Step aboard" button shows the `bujikuibu` card (reuse
`renderChengyu` + a continue button) before advancing to Acre. It does **not**
affect any score (scoring is gone).

### Single ending

`renderEnding` drops the three variants for one written ending. The 6-idiom
glossary stays as a closing recap (every idiom will be marked earned).

## Per-scene narrative plan

Prose is uniform Pamir-weight (~5 dense paragraphs) unless noted. Idiom is
shown on the **right** choice; setbacks are pure consequence + a nudge back and
do not pre-name the idiom.

### Venice (prologue) — `bujikuibu`
Full rewrite: grey lagoon, the loaded Venetian glass, the seventeen-year-old's
fear, the old harbor pilot's blessing. The pilot's proverb becomes the carded
idiom, awarded on the single "Step aboard" click. Plants the "first step /
thousand li" image the ending pays off. Single path, no setback.

### Acre — `yusuzebuda` (narration kept)
- Right: walk with Maffeo (caravan) → idiom card → Hormuz.
- Wrong: board the Genoese galleon → setback: a storm/pirate wreck strands you;
  you wash back to Acre with less than you left. Try again.

### Hormuz — `shibangongbei` (narration kept)
- Right: take the desert guide → idiom card → Pamir.
- Wrong: turn him away → setback: the caravan misreads the Dasht-i Lut, burns
  days at dry streambeds, and is forced back to Hormuz to hire help. Try again.

### Pamir — `saiwengshima` (kept as-is)
Single forced choice (pay the toll). No wrong option. Untouched.

### Kashgar — `jiyuqiucheng` (full rewrite)
The Sunday bazaar, the green-turbaned trader, the cheap brass charm sold as a
shortcut to safe passage. Thread the cast — the desert guide (always with you
now) can warn; Maffeo's patience vs. the lure of a bought guarantee.
- Right: walk on, trust the competence you've earned → idiom card → Khanbaliq.
- Wrong: buy the charm → setback: the charm is a swindle / false comfort that
  leads you to drop your guard; bad luck on the road sends you back. Try again.

### Khanbaliq — `houjibofa` (full rewrite)
The throne room, the Khan old and patient. Callback: the gifts visibly include
what survived the Pamir toll — years of accumulation made tangible. Maffeo/
Niccolò coach restraint; the carded lesson is 厚積薄發. Keep the Khan's existing
`不患無位，患所以立` line as un-carded flavor, distinct from the lesson.
- Right: one gift, one story, one bow → idiom card → ending.
- Wrong: lay everything down at once → setback: the rushed ceremony cools the
  court; an attendant turns you back to compose yourself. Try again.

### Ending — single return (~4–5 paras)
One rich ending: forty-one years old, the repaired leather satchel, the golden
*paiza*, the Khan's `東方不是夢` line kept as flavor, the Genoa prison and
*Il Milione*. Bookend Venice: the thousand-li journey was only ever the sum of
single steps. Then the 6-idiom glossary as a closing recap.

## Execution sequence (Approach A)

**Phase 1 — mechanical skeleton (verify it runs):**
1. Add `bujikuibu` to `CHENGYU`.
2. Add the setback choice model to the click handler + a setback interstitial
   renderer with a "Try again" button that re-renders the current scene.
3. Wire Acre/Hormuz wrong choices with placeholder setbacks; wire Kashgar →
   `jiyuqiucheng` and Khanbaliq → `houjibofa` (right/wrong choices, placeholder
   setbacks).
4. Hook the prologue to award `bujikuibu`.
5. Remove scoring; collapse `renderEnding` to a single ending; expand `allKeys`
   to 6.
6. Keep existing thin prose in place. Verify the game loads and every wrong
   choice loops back. (See Verification.)

**Phase 2 — prose, scene by scene:**
Draft each scene's prose in a side `.txt` file, get approval, then splice into
`index.html` via byte-safe Python (per CLAUDE.md). Order: Venice → Kashgar →
Khanbaliq → ending, plus the Acre/Hormuz setback prose.

## Verification

No JS engine is installed locally (no node/deno/bun), so:
- After every structural edit: byte/quote-escaping checks (all `<span>` markup
  inside double-quoted JS strings must use `\"`; verify CJK bytes, e.g.
  `翁` = `e7 bf 81`).
- After Phase 1 and at each prose splice: ask the user to open `index.html` in a
  browser and click through — confirm all six scenes render, every wrong choice
  shows a setback and loops back, every right choice shows its idiom card, and
  the ending shows the 6-idiom glossary.

## Out of scope

- No new scenes, no reordered scenes, no new choices beyond right/wrong per
  existing decision scene.
- No visual/CSS redesign.
- README's stale chengyu table will be updated as a final cleanup step, not part
  of the core revision.
