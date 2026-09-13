# Fuse

A merge puzzle about agreement. Same ranks fuse into one a rank higher — and it
keeps the charge only if every piece that went into it agreed. Four of a charge
in a run and the lot goes up. Single HTML file, no build step, no dependencies,
no external network requests.

## Controls

| Action | Touch | Keyboard |
|---|---|---|
| Drop into a column | Tap the column | `1`–`5` |
| Pause | Pause button | `Esc` or `P` |
| Mute | Speaker button | `M` |

One tap, and it is the same tap on every difficulty.

## The hook — agreement, not arithmetic

Every cell says two things. **Rank** is the ring around it, one segment per
rank, countable at a glance and readable without numbers. **Charge** is plus,
minus or dead — colour *and* a glyph, so it survives colour blindness and a
bright phone.

Two rules:

> **MERGE.** Dropping a cell fuses the connected run of the same rank it lands
> in — all of it, however big — into one cell a rank higher. It keeps the
> charge only if **every** cell that went into it agreed. One disagreement and
> it comes out dead.
>
> **DETONATE.** A cell that comes out of a merge still charged looks at the
> connected run of its own charge. Four or more, and the whole run goes up.

So merging is how you grow *and* how you ruin a charge, and the thing you are
really arranging is not rank — it is agreement. A careless merge of a plus and
a minus gives you a bigger, deader cell in the middle of the board, which is
worse than the two you had.

A merge has to be **triggered**. Rows arriving from the floor sit where they
land, however much they match; only a cell you dropped, a cell a merge just
made, or a cell that just fell can start one. Ranks stop at 7, and a group of
sevens has nowhere left to go, so it melts down instead.

## The floor

Rows push in from the bottom on a timer **and every so many drops, whichever
comes first**. Fill the top two rows — the dashed line — and the cabinet vents:
two rows blow out and it costs a **fuse**. Out of fuses, out of run.

**A run of seven or more going up at once buys a fuse back**, so the way to earn
safety is to build a bigger chain, never to play smaller.

The second half of the floor rule is not decoration. With the floor on a clock
alone, dropping faster is free — every drop is a chance to merge, a merge of
three or more is a net clear, and an auto-operator tapping twice a second held
an almost empty board for two minutes without trying. Tying the floor to drops
means speed buys nothing, and what you drop and where is the only thing left to
be good at. Both halves tighten as a run goes on.

## Difficulty

One measured number: the **clear budget** — how many cells you have to be
getting rid of, just to stay level.

| | Floor | Budget | Charge bias | Dead pieces | Fuses (max) | Pays |
|---|---|---|---|---|---|---|
| **Cruise** | 9.0s or 8 drops | 0.56 cells/s | 72% | 5% | 4 (6) | ×0.7 |
| **Drive** | 6.5s or 6 drops | 0.77 cells/s | 58% | 10% | 3 (5) | ×1.0 |
| **Redline** | 5.0s or 5 drops | 1.00 cells/s | 46% | 16% | 2 (4) | ×1.45 |

**Charge bias** is how often the next piece agrees with whatever the board
already has most of. At 72% a run of four builds itself if you are paying any
attention; at 46% you have to make one.

**Nothing about the input changes between tiers** — same tap, same column, same
drop. The board, the cells and the queue are **fixed logical sizes**, not
viewport fractions, so a 9:20 phone and a 16:9 desktop play identically; extra
width only buys more panel.

Each tier keeps its own best score, biggest blast and highest rank.

## Playables compliance notes

- **Initial load ~61 KB**, one file. Limit is 30 MB.
- **Zero external requests.** All art drawn procedurally on canvas, all audio
  synthesised with Web Audio. Verified in `test/run.js`.
- **No copyrighted assets** — no image or audio file in the bundle.
- **Scales to 1:1, 16:9 and 9:16.** Screenshots in `test/shots/`.
- **60 fps** at phone and desktop resolutions, measured under load.
- **`firstFrameReady()` then `gameReady()`**, in that order.
- **Pause and mute obeyed immediately.**
- **Progress saved through `saveData` / `loadData`**, localStorage as fallback.
- **No ads wired up yet.**

## Repo layout

```
index.html           the whole game
.nojekyll            serve files as-is
fuse-playables.zip   bundle for the developer portal
src/body.html        source of truth
build.js             wraps src/body.html into index.html
test/driver.js       the auto-operator the other tests share
test/run.js          aspect ratios, external requests, dropping, pause, perf
test/sdk.js          integration against a mocked ytgame SDK
test/gameplay.js     fuse ledger, charge and threshold differentials
test/probe.js        measures how long a run actually lasts
test/bisect.js       disables one draw phase at a time and measures
test/shot.js         screenshot capture
```

`node build.js` rebuilds. `node test/gameplay.js 2` runs one section.

### Testing a puzzle without debug hooks

The shipped build has no test affordances, so the tests play it by looking:
`test/driver.js` samples five points inside every cell — placed outside the
white rank ring and away from the gloss along the top edge, both of which read
as no colour at all — to find where each column tops out and what charge is
sitting there, reads the charge of the next piece out of the queue, and taps a
column. It has three heads: one that lands charges next to charges that agree,
one that just keeps the board flat, and one that does not look at all.

The correctness backbone is an identity. A fuse is spent by exactly one vent
and won back only by a run of seven going up at once, and the run ends when the
last one goes:

```
vents  ===  the tier's starting fuses  +  fuses won
```

Each rule is then tested as a **differential** — the same build with one thing
changed, played by the same operator:

| Claim | Differential |
|---|---|
| a charge only survives a unanimous merge | force it to always survive, and to never survive |
| four is the number | put the threshold out of reach |
| playing well beats playing fast | three operators, same tap rate, one build |

That last one is only a fair test because the floor is tied to drops as well as
to the clock. Tap rate is held equal across all three, so the only variable
left is judgement.

#### The bug that had no board

The first version merged any same-rank run the moment it existed, with no
trigger. The two opening rows are ten rank-one cells, all connected — so they
collapsed into a single cell before the player had touched anything, every
single rise did the same, and the board could never fill. Two minutes of
automated play left nine cells on a forty-cell board and no run ever ended.

A merge needs a trigger. Rows that arrive sit where they land.

## Performance

60 fps at every resolution on the first measurement, because the lessons from
the earlier games in this portfolio were already in place: the cabinet and its
empty wells are baked once per layout at real canvas resolution and blitted
1:1, every cell face is a cached sprite keyed by rank and charge, and no
gradient is built per frame.
