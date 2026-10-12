# AGENTS.md

Single-file, zero-dependency math practice web app (Spanish, 4.º ESO level, for a 15-year-old). The whole app is `index.html` — inline `<style>` + one `<script>` block. No build, no npm, no server. It must stay that way (user requirement: opens by double-click, works offline).

## Commands

There is no package manager. The file *is* the deliverable.

```bash
# run: just open it
open index.html          # macOS

# syntax-check the embedded JS (no browser needed)
python3 -c "
import io,re
s=io.open('index.html',encoding='utf-8').read()
js=re.search(r'<script>(.*?)</script>',s,re.S).group(1)
io.open('/tmp/v.js','w',encoding='utf-8').write(js)"
node --check /tmp/v.js

# test generators under Node (the app tolerates a missing DOM, see below)
cat /tmp/v.js test.js > /tmp/t.js && node /tmp/t.js
```

Always re-extract and `node --check` after editing the `<script>` block — the HTML offers no syntax feedback.

## Architecture

- `RE`, `PO`, `EC` (lines ~105/214/320): arrays of **10 generator functions each** (difficulty 1→10).
- A generator returns `{en, ok, d:[...distractors], ex, pi}`. `en` = statement, `ok` = correct answer string, `ex` = 2–4 line worked explanation (may contain `<b>`/`<br>`), `pi` = hint shown after the *first* wrong answer.
- `withOptions(g)` (line ~422) turns `ok` + `d` into 5 shuffled **unique** options plus a numeric/string padding fallback. If `d` yields fewer than 4 unique distractors you get filler options like `"x = 0 (1)"` — always supply ≥4 real distractors.
- `TEMAS` (line ~416) maps key → name/icono/desc/`niveles`.
- UI is rendered by string-building into `#app` (`renderInicio`, `renderPregunta`, `renderFinal`) with a single delegated `click` handler on `[data-accion]`. Game state lives in the single `est` object (line ~462).
- Mechanics spec: 10 questions, 5 options, a first miss only disables that button and reveals the hint (does **not** end the run), a second miss on the same exercise ends the test.

## Gotchas

- **Unicode escapes**: use `\uXXXX` or a literal character, never `\U000XXXXX` (capital U) — JS silently prints the literal text `U0001f4da`. Astral emoji (🔢 🧩 ⚖️ 📚 🎉 🔁 🏠 🏁) are written as literal UTF-8, not escapes.
- The `<script>` block is deliberately DOM-free at load time: `app` is null-guarded and everything below line 550 is wrapped in `if(typeof document!=="undefined")`. Preserve that guard or the file stops being Node-testable.
- Editing this file with regex/sed is hazardous: strings mix literal UTF‑8 (`x²`, `→`) with `\u00e1`-style escapes, so text-match replacements often fail silently. Re-read the target lines before replacing, and prefer replacing whole statements/blocks.
- No template literals — the codebase is ES5 string concatenation (`"a"+b+"c"`). Match that style; there are no backticks in the file.

## Adding or fixing exercises

1. Add/modify a generator inside the right `RE`/`PO`/`EC` level; keep the answer mathematically exact (answers are displayed, not evaluated).
2. Guard degenerate values that produce ugly or ambiguous output: coefficient `1` (`1x`), `+ 0`, roots of `0`, divisions that are not exact, and decimals like `0.3333333333333333`.
3. Format with the existing helpers: `tCoef`, `tConst`, `quad0`, `polyStr`, `mono`, `eqD`, `eq2`, `sysD`, `sys3D`, `lin2`, `lin3`, `decN`, `F`/`fStr`.
4. Verify: extract JS, `node --check`, then run each generator many times asserting 5 unique options, `0 <= correcta <= 4`, and no `NaN`/`undefined`/double spaces in any option.
