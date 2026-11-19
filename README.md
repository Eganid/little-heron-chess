# Little Heron — a 4 KB chess engine

Entry for the make.fun bounty "Build the strongest chess engine in
4 KB of Python" (tiny-chess-01). MIT licensed; see LICENSE.

## Files

- `main.py` — the whole engine (4,080 bytes), packed by the kit's
  `node cli/makefun.mjs compress` from a readable Python source that
  the submission's transcript bundle carries beside the sessions.
- `README.md` — this file.

## How it plays

Runs as `python3 -P -s /bot/main.py` (Python 3.13, standard library
only) and talks to the contest env at `MAKEFUN_ENV_URL`: `POST /game`
serves the game in play, `POST /move {game, move}` plays a move in
UCI and the house engine answers. The game answer's `fen`,
`legal_moves`, `start_fen` and `moves` drive everything; the chosen
move is always checked against the env's own `legal_moves` before it
is sent. The search runs to a fixed budget of 55,000 nodes a move —
no clock, no randomness — so a replay plays the same moves. On the
kit's practice gauntlet (six house engines, depth-2 to nodes-500k)
it scored 4 wins and 2 losses, a practice Elo of 1,451.

## Design

64-square mailbox board with precomputed knight/king targets and
sliding rays; Zobrist hashing from a fixed seeded LCG; pseudo-legal
move generation with make/unmake and a king-safety test (perft
verified: start position 197,281 at depth 4, Kiwipete 97,862 at
depth 3, and the standard en-passant/pin/promotion positions; legal
moves cross-checked against python-chess on 30 random games).

Negamax search with alpha-beta and principal-variation search, a
transposition table with mate-score adjustment kept across moves,
null-move pruning, check extensions, late-move reductions, killer
move ordering, captures ordered by MVV-LVA, delta pruning, and
quiescence folded into the search: at depth 0 it stands pat over
captures and promotions, and a checked side at the horizon gets a
full evasion ply.
Repetitions and the fifty-move rule score 0 inside the search, as
the house engines score them. The en passant square is hashed only
when an enemy pawn stands beside the pushed pawn, as the env
defines position identity.

Evaluation is material plus piece-square tables (pawn 100, knight
320, bishop 330, rook 500, queen 900); the tables are left-right
symmetric, so the source stores files a–d only, and only the
king's table is blended toward its endgame table as the phase
falls. Measured at 24,000–32,000 nodes a second, a full-budget
move takes at most about 2.8 seconds.
