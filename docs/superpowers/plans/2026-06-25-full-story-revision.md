# Full-Story Revision Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Revise the four thin scenes (Venice, Kashgar, Khanbaliq, ending) to Pamir-v5 quality, wire six idioms (one per scene), and replace branch-and-live-with-it choices with a wrong-choice-loops-back setback model and a single ending.

**Architecture:** Single-file browser game in `index.html` (inline CSS/JS, no build, no deps). Approach A: build and verify the mechanical skeleton first (Phase 1), then write prose scene-by-scene on a working game (Phase 2). Each prose scene is drafted in a side `.txt`, approved, then byte-spliced.

**Tech Stack:** HTML + inline vanilla JS. No test framework, no local JS engine (no node/deno/bun).

## Global Constraints

- File is **UTF-8 with CRLF** line endings. Preserve CRLF on every write.
- **Quote-escaping:** any `<span>`/HTML markup embedded inside a double-quoted JS string MUST escape inner quotes as `\"`. An unescaped `"` breaks the entire SCENES script.
- **Edit-tool only for pure-ASCII** old_string/new_string. Any edit whose matched text contains Chinese characters MUST use a byte-safe Python splice (read bytes, targeted replace, write bytes) — the Edit tool has repeatedly mangled Unicode on this file.
- **No local JS engine:** verification = (a) byte/quote grep checks, and (b) a manual browser click-through the user runs. Never claim runtime-verified without the user's click-through.
- Idiom slug order (journey order): `bujikuibu, yusuzebuda, shibangongbei, saiwengshima, jiyuqiucheng, houjibofa`.
- Commit after every task. Branch: `polo-revision/full-story`.

---

## Phase 1 — Mechanical skeleton (keep existing prose, verify it runs)

### Task 1: Add the `bujikuibu` CHENGYU entry

**Files:**
- Modify: `index.html` (CHENGYU object, just after `const CHENGYU = {`)

**Interfaces:**
- Produces: `CHENGYU.bujikuibu` with the same shape as existing entries (`chars`, `pinyin`, `literal`, `meaning`, `origin`, `chars_breakdown[]`). Consumed by `renderChengyu`, `buildTooltipHtml`, and the glossary.

- [ ] **Step 1: Splice the entry via byte-safe Python**

The entry contains Chinese, so use Python (not Edit). Run this script (adjust the path runner to your shell):

```python
data = open('index.html','rb').read()
anchor = b'  const CHENGYU = {\r\n'
assert data.count(anchor) == 1, "anchor not unique"
entry = (
'    bujikuibu: {\r\n'
'      chars:    "不積跬步",\r\n'
'      pinyin:   "bù jī kuǐ bù, wú yǐ zhì qiān lǐ",\r\n'
'      literal:  "Without piling up half-steps, there is no reaching a thousand li.",\r\n'
'      meaning:  "Distance is not crossed in leaps. Every thousand-li road is only ever the sum of single, unglamorous steps — and the first one is the whole journey in miniature.",\r\n'
'      origin:   "From 《荀子·勸學》: 「不積跬步，無以至千里；不積小流，無以成江海。」— Without half-steps, no thousand li; without small streams, no rivers and seas.",\r\n'
'      chars_breakdown: [\r\n'
'        {char:"不", pinyin:"bù", meaning:"not / without"},\r\n'
'        {char:"積", pinyin:"jī", meaning:"to accumulate, pile up"},\r\n'
'        {char:"跬", pinyin:"kuǐ", meaning:"a half-step (one pace)"},\r\n'
'        {char:"步", pinyin:"bù", meaning:"a step, a pace"}\r\n'
'      ]\r\n'
'    },\r\n'
).encode('utf-8')
data = data.replace(anchor, anchor + entry, 1)
open('index.html','wb').write(data)
print("inserted bujikuibu, new size", len(data))
```

Write this script with the **Write tool** (which handles UTF-8 correctly) and run it with `PYTHONUTF8=1`. Do NOT hand-type the Chinese into a `Bash`/`PowerShell` heredoc — the console is cp1252 and will mangle it. The four characters are 不(U+4E0D) 積(U+7A4D) 跬(U+8DEC) 步(U+6B65).

- [ ] **Step 2: Verify bytes and characters**

```bash
PYTHONUTF8=1 python -c "
d=open('index.html','rb').read()
i=d.find(b'bujikuibu:')
assert i!=-1, 'bujikuibu entry missing'
seg=d[i:i+120]
# chars value bytes for 不積跬步:
expect=bytes.fromhex('e4 b8 8d e7 a9 8d e8 b7 ac e6 ad a5')
assert expect in seg, 'chars bytes wrong (跬 should be e8 b7 ac, not a look-alike)'
print('OK: bujikuibu chars bytes correct (跬 = e8 b7 ac)')
"
```
Expected: `OK: bujikuibu chars bytes correct`. Then Read the file region with the Read tool and confirm 跬 renders as 跬 (not a look-alike like 跪).

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "Add bujikuibu (不積跬步) chengyu entry"
```

---

### Task 2: Add the setback choice model + retry renderer

All edits here are pure-ASCII JS logic — use the **Edit tool**.

**Files:**
- Modify: `index.html` — `renderScene`, its choice handler, a new `showSetbackThenRetry`, and the `goTo` call site.

**Interfaces:**
- Produces: `showSetbackThenRetry(paras, sceneId)` — renders consequence paragraphs + a "Try again" button that re-renders the current scene via `goTo(sceneId)`.
- Produces: choice objects may now carry `setback: string[]` (wrong) instead of `chengyu`+`next` (right). Consumed by the choice handler.
- `renderScene(scene, sceneId)` — now takes the scene id so retries know what to re-render.

- [ ] **Step 1: Give `renderScene` the scene id (Edit tool)**

old_string:
```js
function renderScene(scene) {
```
new_string:
```js
function renderScene(scene, sceneId) {
```

- [ ] **Step 2: Pass the id from `goTo` (Edit tool)**

old_string:
```js
    document.getElementById("scroll").dataset.scene = id;
    renderScene(scene);
```
new_string:
```js
    document.getElementById("scroll").dataset.scene = id;
    renderScene(scene, id);
```

- [ ] **Step 3: Remove the now-unused pending vars (Edit tool)**

old_string:
```js
    let pendingChengyuHtml = "";
    let pendingChengyuKey = null;

    const choicesHtml = scene.choices.map((c, i) => {
```
new_string:
```js
    const choicesHtml = scene.choices.map((c, i) => {
```

- [ ] **Step 4: Replace the choice click handler (Edit tool)**

old_string:
```js
    panel.querySelectorAll(".choice").forEach(btn => {
      btn.addEventListener("click", () => {
        const next = btn.dataset.next;
        const ckey = btn.dataset.chengyu;
        // record score
        if (ckey) state.score += 1;     // wise choice
        else state.score -= 1;          // not the lesson-triggering choice
        if (ckey) {
          // show chengyu card inline, then advance
          pendingChengyuKey = ckey;
          showChengyuThenAdvance(ckey, next);
        } else {
          goTo(next);
        }
      });
    });
```
new_string:
```js
    panel.querySelectorAll(".choice").forEach((btn, i) => {
      btn.addEventListener("click", () => {
        const choice = scene.choices[i];
        if (choice.chengyu) {
          showChengyuThenAdvance(choice.chengyu, choice.next);
        } else if (choice.setback) {
          showSetbackThenRetry(choice.setback, sceneId);
        } else if (choice.next) {
          goTo(choice.next);
        }
      });
    });
```

- [ ] **Step 5: Add `showSetbackThenRetry` after `showChengyuThenAdvance` (Edit tool)**

old_string (the closing of `showChengyuThenAdvance`):
```js
    document.getElementById("afterChengyu").addEventListener("click", () => goTo(next));
  }
```
new_string:
```js
    document.getElementById("afterChengyu").addEventListener("click", () => goTo(next));
  }

  function showSetbackThenRetry(paras, sceneId) {
    const block = document.createElement("div");
    block.className = "setback";
    block.innerHTML =
      `<p class="setback-label">— a wrong turning —</p>` +
      paras.map(p => `<p>${p}</p>`).join("") +
      `<button class="continue-btn" id="afterSetback">Try again</button>`;
    panel.appendChild(block);
    const choicesEl = panel.querySelector(".choices");
    if (choicesEl) choicesEl.style.display = "none";
    block.scrollIntoView({ behavior: "smooth", block: "center" });
    document.getElementById("afterSetback").addEventListener("click", () => goTo(sceneId));
  }
```

- [ ] **Step 6: Verify (byte/quote check)**

```bash
grep -n 'showSetbackThenRetry\|renderScene(scene, id)\|function renderScene(scene, sceneId)' index.html
grep -c 'class="chengyu-tip"' index.html   # expect 0 (no unescaped spans introduced)
```
Expected: the three references found; unescaped-span count is 0. (No behavior change yet — no scene uses `setback` until Task 5. Full browser check happens after Task 5.)

- [ ] **Step 7: Commit**

```bash
git add index.html
git commit -m "Add setback choice model and retry renderer"
```

---

### Task 3: Remove scoring, collapse to a single ending, expand glossary to 6

All pure-ASCII — use the **Edit tool** for every step.

**Files:**
- Modify: `index.html` — `state`, `renderEnding`, its return template, the glossary sub text, and the restart handler.

**Interfaces:**
- Consumes: `CHENGYU.bujikuibu` (Task 1).
- Produces: `renderEnding()` returns a single ending + a 6-entry glossary; no `state.score` anywhere.

- [ ] **Step 1: Drop `state.score` from the state object**

old_string:
```js
    name: "Marco",
    score: 0,                  // wise = +1, unwise = -1 (used for ending variant)
    earnedChengyu: new Set(),  // keys from CHENGYU
```
new_string:
```js
    name: "Marco",
    earnedChengyu: new Set(),  // keys from CHENGYU
```

- [ ] **Step 2: Drop `state.score` reset from the restart handler**

old_string:
```js
      state.name = "Marco";
      state.score = 0;
      state.earnedChengyu = new Set();
```
new_string:
```js
      state.name = "Marco";
      state.earnedChengyu = new Set();
```

- [ ] **Step 3: Replace the variant-selection block with the 6-key glossary list**

old_string:
```js
  function renderEnding() {
    // pick ending variant by score
    let variant;
    if (state.score >= 2)      variant = "humble";       // made wise choices
    else if (state.score >= 0) variant = "trusted";
    else                       variant = "wealthy";

    const variants = {
      humble: {
        title: "Humble Return",
        sub:   "the wisdom of small steps",
        blurb: "You arrive in Venice with very little gold, and a great deal of patience. You are not the loudest man in the tavern, but the merchants seek you out, because the routes you know are the routes that do not lose caravans."
      },
      trusted: {
        title: "Trusted Advisor",
        sub:   "the long acquaintance",
        blurb: "You return with the friendship of the merchant you took into your caravan, and a memory for faces that opens doors from Genoa to Cathay. You are the man a captain wants at his side when the road grows strange."
      },
      wealthy: {
        title: "Wealthy Merchant",
        sub:   "the glittering galleon",
        blurb: "You arrive in Venice with a galleon-full of silk and a story about a shortcut. You live well, and you are never quite sure the shortcut was the right one."
      }
    };
    const v = variants[variant];

    // glossary — show ALL 4 chengyu the player can earn, mark which were earned in this playthrough
    const allKeys = ["yusuzebuda", "shibangongbei", "saiwengshima", "houjibofa"];
```
new_string:
```js
  function renderEnding() {
    // glossary — all six idioms, in journey order (every finisher earns all of them)
    const allKeys = ["bujikuibu", "yusuzebuda", "shibangongbei", "saiwengshima", "jiyuqiucheng", "houjibofa"];
```

- [ ] **Step 4: Replace the variant ending-block in the return template**

old_string:
```js
      <div class="ending-block">
        <h3 class="ending-title">${v.title}</h3>
        <p class="ending-sub">${v.sub}</p>
        <p class="ending-blurb">${v.blurb}</p>
        <button class="continue-btn" id="restart">Begin the journey again</button>
      </div>
```
new_string:
```js
      <div class="ending-block">
        <button class="continue-btn" id="restart">Begin the journey again</button>
      </div>
```

- [ ] **Step 5: Update the glossary subtitle count (3 → 6)**

old_string:
```html
        <p class="sub">Three proverbs from the road, and the small stories behind them.</p>
```
new_string:
```html
        <p class="sub">Six proverbs from the road, and the small stories behind them.</p>
```

- [ ] **Step 6: Verify (grep)**

```bash
grep -nc 'state.score' index.html   # expect 0
grep -n 'v.title\|v.sub\|v.blurb\|variants\[variant\]' index.html   # expect no matches
grep -n 'bujikuibu' index.html | head   # appears in allKeys + Task 1 entry
```
Expected: zero `state.score`, no `v.*`/`variants[variant]` references, `bujikuibu` present.

- [ ] **Step 7: Commit**

```bash
git add index.html
git commit -m "Remove scoring; single ending; 6-idiom glossary"
```

---

### Task 4: Make the prologue award the `bujikuibu` card

Pure-ASCII — **Edit tool**.

**Files:**
- Modify: `index.html` — the prologue's `goAcre` click handler inside `renderScene`.

**Interfaces:**
- Consumes: `showChengyuThenAdvance` (existing), `CHENGYU.bujikuibu` (Task 1).

- [ ] **Step 1: Replace the prologue advance handler**

old_string:
```js
      document.getElementById("goAcre").addEventListener("click", () => goTo(scene.choices[0].next));
```
new_string:
```js
      document.getElementById("goAcre").addEventListener("click", () => {
        document.getElementById("goAcre").style.display = "none";
        const np = panel.querySelector(".prologue-name");
        if (np) np.style.display = "none";
        showChengyuThenAdvance("bujikuibu", scene.choices[0].next);
      });
```

- [ ] **Step 2: Verify (grep)**

```bash
grep -n 'showChengyuThenAdvance("bujikuibu"' index.html   # expect 1 match
```

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "Prologue awards bujikuibu card on departure"
```

---

### Task 5: Wire the four decision scenes to the right/wrong model (placeholders)

Choice labels contain em-dashes (—, non-ASCII), so use a **Python byte-splice**, not the Edit tool. Right choices get `chengyu`+`next`; wrong choices get `setback` (ASCII placeholder, replaced in Phase 2).

**Files:**
- Modify: `index.html` — `choices` arrays of `acre`, `hormuz`, `kashgar`, `khanbaliq`.

**Interfaces:**
- Consumes: the setback handler from Task 2; idiom slugs `jiyuqiucheng`, `houjibofa`.

- [ ] **Step 1: Run the byte-splice (author with Write tool, run with `PYTHONUTF8=1`)**

```python
data = open('index.html','rb').read()

pairs = [
# ---- Acre: board galleon (wrong) -> setback ; walk (right) unchanged ----
('''      choices: [
        { label: "Board the galleon. A gamble on speed — and on the sea not turning.", next: "hormuz" },''',
 '''      choices: [
        { label: "Board the galleon. A gamble on speed — and on the sea not turning.", setback: ["[ACRE SETBACK PLACEHOLDER — replaced in Phase 2]"] },'''),
# ---- Hormuz: turn away (wrong) -> setback ; take him (right) unchanged ----
('''      choices: [
        { label: "Turn him away. The desert chose him, not you.", next: "pamir" },''',
 '''      choices: [
        { label: "Turn him away. The desert chose him, not you.", setback: ["[HORMUZ SETBACK PLACEHOLDER — replaced in Phase 2]"] },'''),
# ---- Kashgar: buy (wrong) -> setback ; walk on (right) -> jiyuqiucheng ----
('''        { label: "Buy it. You've spent silver on worse prayers.", next: "khanbaliq" },
        { label: "Walk on. You have carried yourself this far without help from a cord.", next: "khanbaliq" }''',
 '''        { label: "Buy it. You've spent silver on worse prayers.", setback: ["[KASHGAR SETBACK PLACEHOLDER — replaced in Phase 2]"] },
        { label: "Walk on. You have carried yourself this far without help from a cord.", next: "khanbaliq", chengyu: "jiyuqiucheng" }'''),
# ---- Khanbaliq: one gift (right) -> houjibofa ; all at once (wrong) -> setback ----
('''        { label: "One gift, one story, one bow. Make him wait for each. The Khan has time — you have earned the right to use it.", next: "ending" },
        { label: "Set everything down at once and bow. Perhaps he sees through ceremony. Perhaps he never forgave you for skipping it.", next: "ending" }''',
 '''        { label: "One gift, one story, one bow. Make him wait for each. The Khan has time — you have earned the right to use it.", next: "ending", chengyu: "houjibofa" },
        { label: "Set everything down at once and bow. Perhaps he sees through ceremony. Perhaps he never forgave you for skipping it.", setback: ["[KHANBALIQ SETBACK PLACEHOLDER — replaced in Phase 2]"] }'''),
]

# Normalize the LF in these triple-quoted strings to the file's CRLF before matching.
for old, new in pairs:
    old_b = old.replace('\n', '\r\n').encode('utf-8')
    new_b = new.replace('\n', '\r\n').encode('utf-8')
    c = data.count(old_b)
    assert c == 1, f"expected 1 match, found {c} for: {old[:40]!r}"
    data = data.replace(old_b, new_b, 1)

open('index.html','wb').write(data)
print("wired all four scenes")
```

- [ ] **Step 2: Verify (grep + quote check)**

```bash
grep -n 'setback: \["\[' index.html      # expect 4 placeholder setbacks
grep -n 'chengyu: "jiyuqiucheng"\|chengyu: "houjibofa"' index.html  # expect 1 each
grep -c 'class="chengyu-tip"' index.html  # expect 0 (no broken spans)
```

- [ ] **Step 3: Browser click-through (USER-run — Phase 1 gate)**

Ask the user to open `index.html` and verify:
1. Departure (Venice) shows the 不積跬步 card before advancing to Acre.
2. At Acre, Hormuz, Kashgar, Khanbaliq: the **wrong** choice shows a setback + "Try again" that returns to the same scene; the **right** choice shows the correct idiom card and advances.
3. Reaching the ending shows a single ending + a **six-entry** glossary, all marked earned.
4. "Begin the journey again" restarts cleanly.

Do not proceed to Phase 2 until the user confirms all four.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "Wire Acre/Hormuz/Kashgar/Khanbaliq to right/wrong model (placeholder setbacks)"
```

---

## Phase 2 — Prose, scene by scene

The game now runs end-to-end with placeholder prose/setbacks. Each task below
rewrites one scene's prose to Pamir-v5 quality. **Every prose task is an
approval gate:** draft in a side `.txt`, get the user's sign-off on the words,
THEN splice. The plan fixes the *mechanics and constraints*; the prose is the
deliverable produced and approved at execution time.

### Shared prose-splice procedure (used by Tasks 6–10)

1. Draft the new `narration` array (or setback array) as plain text in
   `scratchpad/<scene>-draft.txt`. Get user approval.
2. Author a Python script with the **Write tool** (handles UTF-8); run with
   `PYTHONUTF8=1`.
3. Match the existing block's exact bytes and replace. Always:
   - Keep **CRLF**: build strings with `\n` then `.replace('\n','\r\n')` before
     `.encode('utf-8')`.
   - Escape every inner `"` in embedded HTML as `\"` inside the JS string.
   - Assert `count == 1` before replacing.
4. **chengyu-tip span format** (for the carded idiom inside narration):
   ```
   <span class=\"chengyu-tip\" data-key=\"SLUG\" tabindex=\"0\"><em>「IDIOM」</em><span class=\"inline-pinyin\">PINYIN</span></span>
   ```
5. Verify after every splice:
   ```bash
   grep -c 'class="chengyu-tip"' index.html   # MUST be 0 (all spans escaped)
   PYTHONUTF8=1 python -c "d=open('index.html','rb').read(); print('CRLF', d.count(b'\r\n'), 'bareLF', d.count(b'\n')-d.count(b'\r\n'))"  # bareLF MUST be 0
   ```
   Then Read the spliced region and confirm Chinese characters render correctly
   (no look-alikes), and ask the user to re-read that scene in the browser.

---

### Task 6: Venice prose rewrite (+ wrap the proverb in a tooltip span)

**Files:** Modify `index.html` — `start.narration`.
**Approval gate:** draft → user sign-off → splice.

**Content constraints (from spec):**
- ~5 dense paragraphs, Pamir-weight. Grey lagoon; the loaded Venetian glass;
  the 17-year-old's fear; the old harbor pilot's blessing.
- The pilot's proverb `不積跬步，無以至千里` must be wrapped in the chengyu-tip
  span with `data-key="bujikuibu"` (so the tooltip works). Pinyin
  `bù jī kuǐ bù, wú yǐ zhì qiān lǐ`.
- Plant the "first step / thousand li" image that the ending pays off.
- Niccolò and Maffeo present and consistent with later scenes.

- [ ] **Step 1:** Draft `scratchpad/venice-draft.txt`; get user approval.
- [ ] **Step 2:** Byte-splice `start.narration` (procedure above), anchoring on
  the current `start:` block's `narration: [` … `]`.
- [ ] **Step 3:** Verify (escaped-span count 0; bareLF 0; Read region; tooltip
  on the proverb works in browser).
- [ ] **Step 4:** Commit — `git commit -m "Rewrite Venice prose; wire bujikuibu tooltip"`.

---

### Task 7: Acre + Hormuz setback prose

**Files:** Modify `index.html` — the two `setback` arrays placed in Task 5.
**Approval gate:** draft both → sign-off → splice.

**Content constraints (from spec):**
- **Acre setback** (chose the galleon): a storm/pirate wreck strands you; you
  wash back to Acre with less than you left. A few paragraphs, then the player
  retries. Do not name the idiom.
- **Hormuz setback** (turned the guide away): the caravan misreads the
  Dasht-i Lut, burns days at dry streambeds, is forced back to Hormuz to hire
  help. Do not name the idiom.

- [ ] **Step 1:** Draft both setbacks in `scratchpad/setbacks-acre-hormuz.txt`; approval.
- [ ] **Step 2:** Byte-splice, replacing each `"[ACRE SETBACK PLACEHOLDER …]"`
  / `"[HORMUZ SETBACK PLACEHOLDER …]"` array element with the approved
  paragraphs (array of strings).
- [ ] **Step 3:** Verify (escaped-span count 0; bareLF 0; browser: each wrong
  choice shows the new setback and loops back).
- [ ] **Step 4:** Commit — `git commit -m "Write Acre and Hormuz setback prose"`.

---

### Task 8: Kashgar prose rewrite (+ setback)

**Files:** Modify `index.html` — `kashgar.narration` and its setback array.
**Approval gate:** draft → sign-off → splice.

**Content constraints (from spec):**
- ~5 dense paragraphs. Sunday bazaar; green-turbaned trader; the cheap brass
  charm sold as a shortcut to safe passage.
- Thread the cast: the desert guide (always present now — deterministic path)
  can warn; Maffeo's patience vs. the lure of a bought guarantee.
- Right choice (walk on) teaches `jiyuqiucheng`; place the chengyu-tip span
  (`data-key="jiyuqiucheng"`, `「急於求成」`, `jí yú qiú chéng`) in the narration
  where Maffeo/the guide names the lesson.
- Setback (bought the charm): the charm is a swindle / false comfort that leads
  you to drop your guard; bad luck sends you back. Do not name the idiom in the
  setback.

- [ ] **Step 1:** Draft `scratchpad/kashgar-draft.txt` (narration + setback); approval.
- [ ] **Step 2:** Byte-splice `kashgar.narration` and the Kashgar setback array.
- [ ] **Step 3:** Verify (escaped-span count 0; bareLF 0; Read region; browser:
  walk-on shows the 急於求成 card, buy-it shows setback + loops back).
- [ ] **Step 4:** Commit — `git commit -m "Rewrite Kashgar prose and setback"`.

---

### Task 9: Khanbaliq prose rewrite (+ setback)

**Files:** Modify `index.html` — `khanbaliq.narration` and its setback array.
**Approval gate:** draft → sign-off → splice.

**Content constraints (from spec):**
- ~5 dense paragraphs. The throne room; the Khan old and patient.
- **Callback:** the gifts visibly include what survived the Pamir toll — years
  of accumulation made tangible.
- **Keep** the Khan's existing `不患無位，患所以立` line as un-carded flavor,
  distinct from the lesson.
- Right choice (one gift, one story, one bow) teaches `houjibofa`; place the
  chengyu-tip span (`data-key="houjibofa"`, `「厚積薄發」`, `hòu jī bó fā`) where
  Maffeo/Niccolò coaches restraint.
- Setback (everything at once): the rushed ceremony cools the court; an
  attendant turns you back to compose yourself. Do not name the idiom.

- [ ] **Step 1:** Draft `scratchpad/khanbaliq-draft.txt`; approval.
- [ ] **Step 2:** Byte-splice `khanbaliq.narration` and the Khanbaliq setback array.
- [ ] **Step 3:** Verify (escaped-span count 0; bareLF 0; Read region — confirm
  both `不患無位` flavor line and the `厚積薄發` carded span are present and
  correct; browser check).
- [ ] **Step 4:** Commit — `git commit -m "Rewrite Khanbaliq prose and setback"`.

---

### Task 10: Ending prose rewrite (single ending, bookend Venice)

**Files:** Modify `index.html` — `ending.narration`.
**Approval gate:** draft → sign-off → splice.

**Content constraints (from spec):**
- ~4–5 paragraphs. Forty-one years old; the repaired leather satchel; the
  golden *paiza*; the Khan's `東方不是夢` line kept as flavor; the Genoa prison
  and *Il Milione*.
- **Bookend Venice:** the thousand-li journey was only ever the sum of single
  steps (echo `bujikuibu`).
- The 6-idiom glossary already follows (Task 3); prose must not duplicate it.

- [ ] **Step 1:** Draft `scratchpad/ending-draft.txt`; approval.
- [ ] **Step 2:** Byte-splice `ending.narration`.
- [ ] **Step 3:** Verify (escaped-span count 0; bareLF 0; browser: ending reads
  as one cohesive return, glossary shows 6 entries).
- [ ] **Step 4:** Commit — `git commit -m "Rewrite ending as a single return"`.

---

### Task 11: Update README chengyu table (cleanup)

**Files:** Modify `README.md` (contains Chinese — Python byte-splice or careful
ASCII-only Edits).

**Content constraints:** The "Locations and their chengyu" and any glossary
summary must match the final code mapping (six idioms, one per scene):
Venice→不積跬步, Acre→欲速則不達, Hormuz→事半功倍, Pamir→塞翁失馬,
Kashgar→急於求成, Khanbaliq→厚積薄發. Remove the stale "covers 3 in full" note.

- [ ] **Step 1:** Update the README tables to the final mapping.
- [ ] **Step 2:** Verify the table matches `allKeys` order in `index.html`.
- [ ] **Step 3:** Commit — `git commit -m "Update README chengyu table to final 6-idiom mapping"`.

---

## Final verification

- [ ] Full browser playthrough: all six scenes Pamir-weight; every wrong choice
  loops back via a written setback; every right choice shows its idiom card;
  Venice tooltip works; ending is one cohesive return + 6-entry glossary.
- [ ] `grep -c 'class="chengyu-tip"' index.html` → 0 unescaped spans.
- [ ] `PYTHONUTF8=1 python -c "d=open('index.html','rb').read(); print(d.count(b'\n')-d.count(b'\r\n'))"` → 0 bare LF.
- [ ] Update `CLAUDE.md`'s scene→chengyu table to the final six-idiom mapping.
- [ ] Offer to merge `polo-revision/full-story` → `main` and push (per the
      finishing-a-development-branch skill).
