# The Travels of Marco Polo — A Chengyu Scroll

A single-file browser game that teaches classical Chinese idioms (成語, chéngyǔ) through Marco Polo's journey from Venice to Kublai Khan's court.

## Play

Open `index.html` in any modern browser. No build step, no dependencies, no server required.

## What it is

A choose-your-own-adventure set across six historical locations. At each stop, you face a fateful decision. The choices introduce HSK 6 / gaokao-level chengyu in context — each one appears in the story narration with inline pinyin, then is reinforced on a dedicated card with its literal meaning, best translation, and classical origin.

Hover over any underlined chengyu phrase to see a character-by-character breakdown in a tooltip.

## Locations and their chengyu

| Scene | Location | Chengyu taught |
|---|---|---|
| 1 | Venice, 1271 | — (prologue) |
| 2 | Acre, the Levant | 急於求成 · 塞翁失馬 |
| 3 | Hormuz, Persian desert | 厚積薄發 |
| 4 | Pamir Mountains | 欲速則不達 |
| 5 | Kashgar bazaar | 事半功倍 |
| 6 | Khanbaliq, Khan's court | — |

A glossary at the end covers 塞翁失馬, 厚積薄發, and 事半功倍 in full.

## Chengyu reference

| Idiom | Pinyin | Meaning |
|---|---|---|
| 事半功倍 | shì bàn gōng bèi | Half the effort, double the result |
| 塞翁失馬 | sài wēng shī mǎ | A loss may be a blessing in disguise |
| 厚積薄發 | hòu jī bó fā | Accumulate deep; release sparingly |
| 急於求成 | jí yú qiú chéng | Impatience that causes its own failure |
| 欲速則不達 | yù sù zé bù dá | He who desires speed does not arrive |

## Technical notes

- Single HTML file (~1,100 lines) — all CSS and JS inline
- Fonts: Press Start 2P (pixel display text), Noto Serif SC (Chinese characters)
- Per-scene color palettes via CSS custom properties on `main[data-scene]`
- No frameworks, no bundler, no runtime dependencies
