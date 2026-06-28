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
- **peek** `?` alone → full system panel (bat/net/cpu/ram/dsk/upt/ip/os as a clean icon-less list); `?bat|net|ram…` → single live readout
- **math** `=`/operators → inline answer (tap to copy) + Calculator tile + routes
- **code** leading `ter/fil/set/mod/mus` → scoped command mode (positional arg stack + Enter to run)
- **plain** → fuzzy apps → actions → web/Files routes (≥3 chars before any check runs)

## Resolve hierarchy (ranked Result[])
answers → app tiles (fuzzy) → action cards → general-search routes (Search + Files). `fuzzy()` scores prefix(3) > substring(2) > subsequence(1).

## Render
- **Pinned** (empty): 64px tiles, grayed TUI code under name
- **App matches**: horizontally-swipeable 64px tile row (~4.25 visible = "more" cue)
- **Cards**: grouped list w/ hairline separators, icon + name
- **Answer**: grey-framed accent row (math/peek)
- **Routes**: query text + Search/Files tiles
- **Action hints** (≥3 chars): grey autocomplete tail per route — `app — open`, `… — search in files`, `… — search web` (icon-less; leading icon only in code mode)
- **Recent recall** (toggle): solid floating overlay window (`#1c1c1c`, shadow, ~3.5 rows) anchored above the bar; older queries fade via a fixed gradient text-layer (`.rov-fade`), selected row crisp. Off → bottom-anchored full reversed-history list.
- **Command args** (Minecraft-style): in code mode an `ARG_TREE` grammar drives positional suggestions one argument at a time (`set ` → wifi/brightness/bluetooth/airplane → `set wifi ` → off/on). Highlighted option = grey bar ghost; typed prefix dims in the stack. Tab / → / tap accept and advance; ↑↓ cycle; Enter runs. Floating stack anchors to the current token's start (stable while typing). **List toggle** → render the same options as icon-less rows in the results list instead of the floating stack.
- **Results fade**: both edges masked (top 22px, bottom 16px) so swipeable tiles don't clip hard against chips/bar
- **Bar**: styled code token (accent), caret at `cursorPos`, ghost autosuggest, app icon when scoped; SVG cross = clear-text / close-surface; `min-height:68px`, `border-radius:18px`

## Interaction
- **Keyboard-agnostic** key handling (hardware + on-screen share one path): Enter (run/launch pinned#1), ↑↓ (select; ↑ on empty = recall), ←→ (caret; → accepts ghost), Tab (accept ghost / first arg / navigate), Esc (close recall), Backspace, type.
- **Chips** (top-bar toggle): 3 live suggestions per mode.
- **Ghost autocomplete**: code-token completion (`te`→`ter`) + terminal-style history autosuggest.
- **On-screen keyboard**: kbd-alpha QWERTY, SVG shift/back/enter glyphs.
- **Spacebar trackpad**: hold/drag space → cursor mode (dims keys, brightens space); ←→ moves caret, ↑↓ = arrow keys, tap = space.
- **Recent searches**: dedupe history (cap 20). Overlay mode — **swipe up on bar** summons it; ↑↓ / spacebar / touch scroll between rows; landing on a query places caret **after the last letter**; **Backspace dismisses** the overlay (doesn't delete). Live results render behind it. Non-overlay mode — recall list via ↑ on empty bar.

## Gestures
- **Open**: swipe up on home — one 0→1 progress; dock rises (phase 1), search slides up + dock recedes (phase 2). `OPEN_DIST=260`.
- **Close**: swipe down on topbar / bar / keyboard, OR overscroll-down at top of results (`__surfDrive`). Direction-aware snap: flick velocity (±0.45 px/ms) or drag threshold (open >0.35, close <0.78).
- Deferred pointer-capture (only after move >4px) so taps stay taps; capture-phase click swallow guards tap-vs-drag.

## Launched-app stubs
Terminal (fake shell: cd/ls/pwd), Files (filtered list), toasts for the rest. `‹ back` / Esc closes.

## Toggles
On-screen↔Connected KB · Chips on/off · Recent overlay on/off (solid window vs full list) · List on/off (command args as results-list rows vs floating stack).

## Figma documentation (file `H1f7PUCNLbEBou7eK62XQH`, section `667:3`)
Short description of each board:
- **User-flow screens** — main normal flows rebuilt to match the prototype: neutral/pinned state, type-`a`-with-results (no keyboard), expanded search, in-app, web search, files.
- **Keyboard component** (`674:5`) — reusable non-interactive QWERTY.
- **Icon images frame** (`680:912`) — real app icons (`camera/terminal/music/files/notes/settings-64`) used as IMAGE fills.
- **Search bar component** — variant set documenting every bar state (empty, typing, code token, ghost, scoped icon) for the design system.
- **Code-arg explorations** — settings flow with Minecraft-style stacked args rising as you type + gray autofill; a "many options" variant with scroll-gradient + reddish semantic reset/disable (sudo → password); files flow (no sudo); plus a list-format version rendering args as normal search rows.
- **Recent-search overlay frame** (`698:3168` → panel `710:2`) — solid clipping window (260×132, `#161616`, radius 18) with rows + inner gradient fade layer.
