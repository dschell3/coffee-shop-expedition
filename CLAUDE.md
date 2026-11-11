# Coffee Shop Expedition: Game v2

A playable web version of the Coffee Shop Expedition, a lattice-path game for a MATH 101 group project at Sacramento State. The audience is high school students, plus classmates and the instructor reviewing the project. v2 adds the rule variations from our write-up.

## Repo layout

```
/index.html        the live page. Kept as an exact copy of /v2/index.html.
/v2/index.html     v2, the file we edit
/CLAUDE.md         this file
```

- Hosted on GitHub Pages from `main`: `https://dschell3.github.io/coffee-shop-expedition/` (and the same page at `.../coffee-shop-expedition/v2/`).
- The printed QR code in our write-up points to the root URL. Edit `/v2/index.html`, then copy it over `/index.html` in the same commit so the two never drift.

## Hard constraints

- **One self-contained HTML file.** Inline CSS and vanilla JS. No frameworks, no build step, no npm, no external requests. It must work by double-clicking the file with no internet.
- **Phone first.** Reviewers play it on their phones in class. Test at about 400px wide: no sideways scrolling, tap targets at least 44px, and the board scales to fit.
- **Light and dark mode.** Every color is a CSS custom property on `:root`, redefined under `@media (prefers-color-scheme: dark)`. SVG elements get colors through classes or `style="fill:var(--x)"`, never hard-coded hex.
- **Accessible.** Keyboard play (arrow keys), visible focus, `prefers-reduced-motion` respected, an `aria-label` on the board SVG, and an `aria-live` turn status.
- **Plain language in all UI text.** Short sentences, small words, high school level. No jargon without a one-line explanation. Don't try to sound smart.
- The goal corner is labeled "Starbucks" to match the class worksheet. Use a generic coffee-cup icon, never a logo.

## How the code works

- State: `mode`, `n, m` (board width and height; Starbucks is at corner `(n, m)`), `path` (array of `{x, y}` corners), `moves` (array of `{d, k}`: direction `'N' | 'E' | 'S' | 'W'` and how many blocks), and `setup` (true while Starbucks can still be placed).
- `legalMoves()` lists every move from the current corner under the current mode. `move(d, k)`, `undo()`, `newGame()`, `setBoard(w, h)`, `setMode(key)`, then `render()` redraws everything: the SVG board, turn panel, route word, move log, and scoreboard.
- The board is SVG built as a string. Moves are colored by player (P1 green, P2 orange). Glowing dashed circles mark the legal next corners and can be tapped.
- **Board setup.** There are no board presets. Default is 3×3. At page load, and after "Change board", a faint 6×6 map appears and the player taps any corner to put Starbucks there (Width and Height steppers do the same for keyboard users). Setup ends with the first move. "Play again" keeps the board and skips setup. Railroad boards are always square.
- `countPaths(w, h, ok)` is the corner-rule DP. `makeSolver()` says who wins from any corner in modes without backward moves. Both are reused by hints, the rules box, and the self-check.
- Keys: ↑ North, → East, ↓ South and ← West on chaos turns, 1–9 block count, H hint, U undo, R play again, B change board.
- The side cards (Route so far, Move log, Scoreboard, How to play) are collapsible. Route so far starts open. The turn card is always open.

## Modes (built)

The mode picker is a row of pills above the board. Each mode shows a one or two sentence rule summary under the board, plus the route count for the board. The scoreboard is kept per mode and board size.

### Classic
Take turns moving 1 block North or East. Whoever reaches Starbucks wins.

### Long moves
On your turn, move any number of blocks (at least 1), but all in one direction: all North or all East. Can't go past the edge.
- Tapping any corner straight up or straight right of the token is a legal move. All are highlighted.
- Math: this is two-pile Nim. With `a` blocks left East and `b` left North, the player to move is losing exactly when `a == b`. The winning move is to make them equal, then copy the opponent.
- "Hint" button: shows the winning move if there is one, or says "No winning move. Try to make them slip up." Also available in Railroad.

### Railroad
Square boards only. A track runs along the diagonal from Start to Starbucks. The token may touch the track but never cross below it. Legal corners have `y >= x`.
- The track is a dashed line on the diagonal. Corners below it are dimmed and never offered as moves.
- Math: the number of legal paths on an n×n board is the Catalan number: 1, 1, 2, 5, 14, 42 for n = 0 to 5.
- With 1-block moves every game takes 2n moves, so Player 2 always wins. A "Long moves" toggle makes choices matter.

### Chaos
Long-move rules, and on every 3rd turn of the game (turns 3, 6, 9, and so on) the player may step back South or West instead. Can't leave the board, and can't step back onto Start.
- On chaos turns a "Chaos turn! You may step back" banner shows and the backward corners are drawn dotted.
- **Open design question: termination.** Backward moves can make the game loop forever. Three options sit behind a dropdown so the group can compare them. Default is option 1 until the group decides. Once decided, keep that one and delete the dropdown.
  1. Backward moves are capped at 1 block (forward moves can still be long).
  2. Each player gets 2 chaos moves per game, shown as coffee-cup tokens.
  3. A 30-turn limit. If it runs out, nobody wins.
- The old 1-block-only Chaos mode was built and then removed because it played the same as Classic (the parity argument below). Don't bring it back.

### After a board's first game
The rules box adds "With best play, Player X wins on this board" (from the solver). It only shows once that board has a finished game in that mode, so the first game stays a surprise. Chaos never shows it, since moves can loop.

## Puzzles (built)

A 5th pill in the mode row, "Puzzles". One player, drawn in Player 1's color. No auto-opening intro screen. Level pills 1, 2, 3 sit in the turn card, with a "Reset this level" button. Each level fixes its own boards (the player can't change them), and leaving Puzzles brings back the board the player had. Progress lives in memory for the session only.
1. **Find every path** on 2×2, then 3×2. Trace a route with taps or arrow keys. Reaching Starbucks records it as a word (NNEE) and clears the board. Repeats are rejected. Found routes are drawn faintly. Shows "4 of 6 found." On completion, shows "4 choose 2 = (4 × 3) ÷ (2 × 1) = 6" (built by `chooseText`, never typed in).
2. **Fill in the corners** on 3×3. The bottom row and left column are prefilled with 1s. A corner shows "?" once the corners below it and to its left are known. Tapping it (or Enter on the focused box) opens a number field. Right answers lock in. Wrong ones say "Add the corner below (b) and the corner to the left (l)." Ends with "20 routes reach Starbucks. Same as 6 choose 3."
3. **Railroad** on 2×2, 3×3, 4×4 with the track. Same fill-in, but corners under the track are off limits and count 0. Ends with "2, 5, 14. These are called Catalan numbers."

The write-up's "Roadwork" level (closed streets) is out of scope. The game block is done: don't add modes, settings or levels beyond this file.

## Math facts to keep correct

- Paths on an n×m board: C(n + m, m). 2×2 = 6, 3×2 = 10, 3×3 = 20, 5×4 = 126.
- Corner rule: paths(x, y) = paths(x − 1, y) + paths(x, y − 1), with 1s along the bottom row and left column.
- Classic winner: if n + m is odd, Player 1 wins. If even, Player 2 wins.
- Long moves: positions with equal blocks left East and North are losses for the player to move.
- Catalan numbers: 1, 1, 2, 5, 14, 42 (n = 0 to 5).
- Chaos with 1-block moves keeps the same parity, so the same winner. (Long forward moves break this, which is why Chaos uses them.)

If you add a counter or hint, compute it in code (for example with a DP over corners). Never hard-code the answer. The self-check `<script>` at the end of the file runs in the console and asserts the values above. Extend it when you add math.

## Testing before handing back

- Play at least one full game in every mode on a few board sizes, including 1×1 and 6×6: win message, undo, play again, change board, keyboard play.
- Check at 400px and at desktop width, in both light and dark mode.
- Check there are no console errors, and that the self-check reports all passed.
- Confirm `/index.html` and `/v2/index.html` are identical (`diff index.html v2/index.html`).

## Git workflow

- **Branch:** do all v2 work on `darren/v2`. Create it from `main` if it doesn't exist (`git checkout -b darren/v2`). Never commit to `main` directly. Darren merges when v2 is ready.
- **Author:** every commit is in Darren's name with his GitHub ID, **dschell3**. Before the first commit, check `git config user.name` and `git config user.email`. If they aren't Darren's, set them for this repo only:
  ```
  git config user.name "Darren"
  git config user.email "<Darren's GitHub noreply email, from GitHub Settings > Emails>"
  ```
  Ask Darren for the email if it isn't already set. Don't guess it.
- **No signing, no AI attribution:** run `git config commit.gpgsign false` for this repo so commits aren't signed with a key GitHub can't verify. Don't add `Co-Authored-By`, "Generated with", or similar lines to commit messages or PRs.
- **Small commits:** one step per commit, each leaving the game playable. Each commit that changes `/v2/index.html` also copies it over `/index.html`.
- **Commit messages:** short, plain, and present tense, for example "Add long moves mode" or "Fix railroad check on 4x4 board".

## Working style

- Darren prefers short, plain explanations. Ask before big design changes, and say what you changed in a few sentences. For anything bigger than a fix, lay out a commit plan first and wait for a go.
