# Coffee Shop Expedition: Game v2

A playable web version of the Coffee Shop Expedition, a lattice-path game for a MATH 101 group project at Sacramento State. The audience is high school students, plus classmates and the instructor reviewing the project. v2 adds the rule variations from our write-up.

## Repo layout

```
/index.html        v1 (rough prototype). FROZEN: do not edit until Darren says so.
/v2/index.html     v2, the version we're building now
/CLAUDE.md         this file
```

- Hosted on GitHub Pages: v1 at `https://dschell3.github.io/coffee-shop-expedition/`, v2 at `.../coffee-shop-expedition/v2/`.
- The printed QR code in our write-up points to the root URL. When v2 is done, Darren will copy it over `/index.html`. Until then, never touch the root file.
- Start v2 from a copy of v1 (`/index.html`) and build on it.

## Hard constraints

- **One self-contained HTML file.** Inline CSS and vanilla JS. No frameworks, no build step, no npm, no external requests. It must work by double-clicking the file with no internet.
- **Phone first.** Reviewers play it on their phones in class. Test at about 400px wide: no sideways scrolling, tap targets at least 44px, and the board scales to fit.
- **Light and dark mode.** Keep v1's approach: every color is a CSS custom property on `:root`, redefined under `@media (prefers-color-scheme: dark)`. SVG elements get colors through classes or `style="fill:var(--x)"`, never hard-coded hex.
- **Accessible.** Keyboard play (arrow keys), visible focus, `prefers-reduced-motion` respected, an `aria-label` on the board SVG, and an `aria-live` turn status.
- **Plain language in all UI text.** Short sentences, small words, high school level. No jargon without a one-line explanation. Don't try to sound smart.
- The goal corner is labeled "Starbucks" to match the class worksheet. Use a generic coffee-cup icon, never a logo.

## How v1 works (what you're building on)

- State: `n, m` (board width and height), `path` (array of `{x, y}` corners), and `moves` (array of `'N'`/`'E'`).
- `move(d)`, `undo()`, `newGame()`, `setSize(key)`, then `render()` redraws everything: the SVG board, turn panel, route word, move log, and scoreboard.
- The board is SVG built as a string. Moves are colored by player (P1 green, P2 orange). Glowing dashed circles mark the legal next corners and can be tapped.
- Board presets: 3×2, 3×3, 5×4.
- Keys: ↑ North, → East, U undo, R restart.

## Modes and rules to build

Let the player pick a **mode** next to the board-size picker. Each mode shows a one or two sentence rule summary under the board. Keep the scoreboard per mode and board size.

### 0. Classic (v1 behavior, keep as is)
Take turns moving 1 block North or East. Whoever reaches Starbucks wins.

### A. Long moves
On your turn, move any number of blocks (at least 1), but all in one direction: all North or all East. Can't go past the edge.
- UI: tapping any corner straight up or straight right of the token is a legal move. Highlight them all.
- Math: this is two-pile Nim. With `a` blocks left East and `b` left North, the player to move is losing exactly when `a == b`. The winning move is to make them equal (get back on the diagonal), then copy the opponent.
- Optional "Hint" button: show the winning move if there is one, or say "No winning move. Try to make them slip up."

### B. Railroad tracks
Square boards only (3×3, 4×4, 5×5). A track runs along the diagonal from Start to Starbucks. The token may touch the track but never cross below it. Legal corners have `y >= x`.
- Draw the track as a dashed line on the diagonal and hide or disable illegal moves.
- Math: the number of legal paths on an n×n board is the Catalan number: 1, 1, 2, 5, 14, 42 for n = 0 to 5.
- Note: with 1-block moves, every game still takes 2n moves, so Player 2 always wins. Offer **B + long moves** as a toggle, since that combination makes choices matter.

### C. Coffee House Chaos
Classic rules, plus on every 3rd turn of the game (turns 3, 6, 9, and so on) the player may move 1 block South or West instead. Can't leave the board, and can't move back onto Start.
- UI: on chaos turns, show a clear "Chaos turn! You may step back" banner and highlight the backward options in a different style.
- Math, and the surprise to surface in the UI after a few games: every move changes x + y by exactly 1, so the total number of moves always has the same odd or even parity as n + m. The winner is still decided before the game starts. Show this in an end-of-game "Why?" note.
- The game always ends: each 3 turns give at least 2 steps forward and at most 1 back.

### D. Chaos + Long moves (experimental)
Long-move rules, and on chaos turns the player may move South or West instead. This breaks the parity lock, so choices matter.
- **Open design question: termination.** If backward moves can be long, the game can loop forever. Prototype these options behind a setting and let us compare:
  1. Backward moves are capped at 1 block (forward moves can still be long).
  2. Each player gets a limited number of chaos moves per game (default 2), shown as coffee-cup tokens.
  3. A turn limit (for example 30). If it runs out, nobody wins.
- Default to option 1 until we decide.

### Puzzle Mode (1 player, nice to have)
Levels that walk through the math in our write-up:
1. **Find every path** on 2×2, then 3×2. Trace a path. It's recorded as a word (NNEE). Repeats are rejected. Show "4 of 6 found." On completion, show "4 choose 2 = (4 × 3) ÷ (2 × 1) = 6."
2. **Fill in the corners.** The player types how many paths reach each corner. Check each one with the "below + left" rule.
3. **Roadwork.** Some streets are closed. Count the paths with the corner rule.
4. **Railroad.** Count the paths that stay on or above the track: 2, 5, 14.

## Math facts to keep correct

- Paths on an n×m board: C(n + m, m). 2×2 = 6, 3×2 = 10, 3×3 = 20, 5×4 = 126.
- Corner rule: paths(x, y) = paths(x − 1, y) + paths(x, y − 1), with 1s along the bottom row and left column.
- Classic winner: if n + m is odd, Player 1 wins. If even, Player 2 wins.
- Long moves: positions with equal blocks left East and North are losses for the player to move.
- Catalan numbers: 1, 1, 2, 5, 14, 42 (n = 0 to 5).
- Chaos with 1-block moves keeps the same parity, so the same winner.

If you add a counter or hint, compute it in code (for example with a DP over corners). Never hard-code the answer. Add a quick self-check in a `<script>` that runs in the console and asserts the values above.

## Testing before handing back

- Play at least one full game in every mode and board size: win message, undo, restart, keyboard play.
- Check at 400px and at desktop width, in both light and dark mode.
- Check there are no console errors.
- Confirm the root `/index.html` is unchanged (`git diff --stat` should only show `v2/`).

## Git workflow

- **Branch:** do all v2 work on `darren/v2`. Create it from `main` if it doesn't exist (`git checkout -b darren/v2`). Never commit to `main` directly. Darren merges when v2 is ready.
- **Author:** every commit is in Darren's name with his GitHub ID, **dschell3**. Before the first commit, check `git config user.name` and `git config user.email`. If they aren't Darren's, set them for this repo only:
  ```
  git config user.name "Darren"
  git config user.email "<Darren's GitHub noreply email, from GitHub Settings > Emails>"
  ```
  Ask Darren for the email if it isn't already set. Don't guess it.
- **No AI attribution:** don't add `Co-Authored-By`, "Generated with", or similar lines to commit messages or PRs.
- **Small commits:** one step per commit, each leaving the game playable. A suggested order:
  1. Copy v1 to `v2/index.html` (no changes)
  2. Add the mode picker and rule summary (Classic only)
  3. Mode A: long moves
  4. Nim hint button
  5. Mode B: railroad tracks
  6. B + long moves toggle
  7. Mode C: Coffee House Chaos
  8. Chaos "Why?" end-of-game note
  9. Mode D: Chaos + long moves, with the termination setting
  10. Math self-check script
  11. Puzzle Mode levels, one commit per level
- **Commit messages:** short, plain, and present tense, for example "Add long moves mode" or "Fix railroad check on 4x4 board".

## Working style

- Darren prefers short, plain explanations. Ask before big design changes, and say what you changed in a few sentences.
