# Mecha Comet — Feature Reference

Single-file HTML prototype (`index.html`). Device frame **540×620**, AMOLED-black, Sora font. The search screen *is* the OS home. No build step — serve static (`npx http-server`).

## Architecture
Input string → **parseInput** → **resolve** (ranked) → **render**. All behavior is config-driven by data maps, not hardcoded branches.

| Map | Role |
|---|---|
| `ACCENTS` | per-app accent color |
| `ICONS` | app id → icon image path (else 3-letter text tile) |
| `CODES` | TUI codes `ter/fil/set/mod/mus` → {app, appId, accent, demo args} |
| `CODE_BY_APP` | reverse: app id → code (for grayed code under tiles) |
| `APPS` | searchable app registry (fuzzy launcher) |
| `ACTIONS` | in-app sub-features ("App › feature") |
| `PINNED` | 4 empty-state pinned apps; #1 = implicit Enter target |

## Layers (z-stacked in `.phone`)
1. `#homeLayer` — home wallpaper, swipe-up to open
2. `#dockLayer` — nav dock, rises mid-gesture
3. `#searchSurface` — the search UI (topbar, results, chips, bar, keyboard, app overlay, toast)

## Input modes (parseInput)
- **empty** → pinned grid
- **peek** `?bat|net|ram|cpu` → live system readout
- **math** `=`/operators → inline answer (tap to copy) + Calculator tile + routes
- **code** leading `ter/fil/set/mod/mus` → scoped command mode (args + Enter to run)
- **plain** → fuzzy apps → actions → web/Files routes

## Resolve hierarchy (ranked Result[])
answers → app tiles (fuzzy) → action cards → general-search routes (Search + Files). `fuzzy()` scores prefix(3) > substring(2) > subsequence(1).

## Render
- **Pinned** (empty): 64px tiles, grayed TUI code under name
- **App matches**: horizontally-swipeable 64px tile row (~4.25 visible = "more" cue)
- **Cards**: grouped list w/ hairline separators, icon + name
- **Answer**: grey-framed accent row (math/peek)
- **Routes**: query text + Search/Files tiles
- **Recent recall**: bottom-anchored reversed history list
- **Bar**: styled code token (accent), caret at `cursorPos`, ghost autosuggest, app icon when scoped; ✕ = clear-text / ⌄ = close-surface

## Interaction
- **Keyboard-agnostic** key handling (hardware + on-screen share one path): Enter (run/launch pinned#1), ↑↓ (select; ↑ on empty = recall), ←→ (caret; → accepts ghost), Tab (accept ghost / first arg / navigate), Esc (close recall), Backspace, type.
- **Chips** (top-bar toggle): 3 live suggestions per mode.
- **Ghost autocomplete**: code-token completion (`te`→`ter`) + terminal-style history autosuggest.
- **On-screen keyboard**: kbd-alpha QWERTY, SVG shift/back/enter glyphs.
- **Spacebar trackpad**: hold/drag space → cursor mode (dims keys, brightens space); ←→ moves caret, ↑↓ = arrow keys, tap = space.
- **Recent searches**: dedupe history (cap 20), recall panel via ↑ on empty bar.

## Gestures
- **Open**: swipe up on home — one 0→1 progress; dock rises (phase 1), search slides up + dock recedes (phase 2). `OPEN_DIST=260`.
- **Close**: swipe down on topbar / bar / keyboard, OR overscroll-down at top of results (`__surfDrive`). Direction-aware snap: flick velocity (±0.45 px/ms) or drag threshold (open >0.35, close <0.78).
- Deferred pointer-capture (only after move >4px) so taps stay taps; capture-phase click swallow guards tap-vs-drag.

## Launched-app stubs
Terminal (fake shell: cd/ls/pwd), Files (filtered list), toasts for the rest. `‹ back` / Esc closes.

## Toggles
On-screen↔Connected KB · Chips on/off.
