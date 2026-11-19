# tiny-chess-01 — a chess engine in 4,069 bytes of Python

`main.py` is the whole engine. It is an lzma-compressed, base85-encoded
payload with a two-line unpacker (`import lzma,base64; exec(...)`) — the
same packing the rules explicitly allow and the house baseline uses. The
unpacked source is ~8 KB of golfed Python 3.13, standard library only. It
reads, imports and runs no other file.

## How it plays

The program talks to the env at `MAKEFUN_ENV_URL`: `POST /game` serves
the game in play, `POST /move {game, move}` plays a move in UCI, and the
loop ends when the env answers `{"status": "done"}`. Before every move
it rebuilds the position by replaying `start_fen` + `moves`, so its
repetition history is always exact.

## The engine

- 0x88 board, pseudo-legal move generation filtered by king safety,
  incremental make/unmake with zobrist key, material score, game phase
  and bishop counts maintained move by move.
- Negamax alpha-beta with a transposition table (exact/lower/upper),
  check extensions, and quiescence search over captures and promotions
  (all evasions when in check). Repetitions and the fifty-move rule
  score 0 inside the search, like the house engines.
- Move ordering: TT move, MVV-LVA captures, promotions, two killer
  moves per ply.
- Evaluation: material (100/320/330/500/900) plus piece-square tables
  for every piece — stored file-symmetrically, 32 characters a table —
  a king table blended between middlegame and endgame by material
  phase, and the bishop pair.
- Deterministic by construction: iterative deepening under a fixed
  budget of 40,000 nodes a move. No clock is ever read and there is no
  randomness (the zobrist table comes from a fixed-seed PRNG), so a
  replayed trial plays the same moves. At ~40k nodes/second the slowest
  move is about a second, far under the 10 s limit, and a whole trial
  takes minutes of the 1,800 s wall.

## Local result

On the kit's practice trial 0 (6 games, run against the kit's own env
server): wins by checkmate against depth-2, depth-3 and depth-4,
losses by checkmate against nodes-100k, nodes-250k and nodes-500k —
3.0 points, a practice Elo of **1279** (final floor: 1154; the sunfish
house baseline rates ~1365 on practice trials). Move generation was
verified against published perft values on five standard positions.
