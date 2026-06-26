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
| 1 | Venice, 1271 | 不積跬步 (prologue tooltip) |
| 2 | Acre, the Levant | 欲速則不達 |
| 3 | Hormuz, Persian Gulf | 事半功倍 |
| 4 | Pamir Mountains | 塞翁失馬 |
| 5 | Kashgar bazaar | 僧多粥少 |
| 6 | Khanbaliq, Khan's court | 厚積薄發 |
| 7 | Ending / the long road home | — (glossary) |

All six idioms are reviewed in a full glossary at the end.

## Chengyu reference

| Idiom | Pinyin | Meaning |
|---|---|---|
| 不積跬步 | bù jī kuǐ bù | Without half-steps, no thousand-li road |
| 欲速則不達 | yù sù zé bù dá | He who desires speed does not arrive |
| 事半功倍 | shì bàn gōng bèi | Half the effort, double the result |
| 塞翁失馬 | sài wēng shī mǎ | A loss may be a blessing in disguise |
| 僧多粥少 | sēng duō zhōu shǎo | Many monks, little porridge — more demand than supply |
| 厚積薄發 | hòu jī bó fā | Accumulate deep; release sparingly |

## Technical notes

- Single HTML file (~1,000 lines) — all CSS and JS inline
- Fonts: Press Start 2P (pixel headers), VT323 (UI labels), Noto Serif SC (body and Chinese characters)
- Per-scene color palettes via CSS custom properties on `main[data-scene]`
- No frameworks, no bundler, no runtime dependencies
