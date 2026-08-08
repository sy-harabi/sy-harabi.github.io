---
layout: single
title: "Build Your Own Combat Blob: An Implementation Guide"

toc: true
toc_sticky: true

comments: true
---

I wrote a [research log]({% post_url 2026-06-16-Theorycrafting-a-Combat-Blob-A-Research-Log %}) about building a combat blob a while back. That post was the *why*: the story, the dead ends, the ideas that looked clever and weren't. This one is the *how*. If you've read the log and want to actually start building, or if you just landed here cold, this is the architecture and the handful of decisions that carry the whole thing.

One warning before we start: none of this is *the* answer. It's one design that works, and most of what makes it work is a few load-bearing ideas you could reach by a completely different road. Take the ideas, not the code. Where I show pseudocode it's to pin down a concept, never to hand you something to paste.

## What we're actually building

A **blob** is the least structured attack force in Screeps: a loose pack of combat creeps, no formation, that you shove at a defended base and hope survives. It sounds like it needs no code at all. It needs a surprising amount.

Every tick you pick a move for every creep, and three goals pull against each other:

- **Survive** — don't let a single creep die to focused fire.
- **Advance** — close the distance to whatever you came to kill.
- **Park** — once in range, hold still and keep as many attackers shooting as possible.

Advancing walks you into tower fire that only gets worse the closer you get. Parking deep is where you do damage and also where you die. And the enemy is a black box — you don't get to know who they'll shoot.

That last point is the first real design decision, so make it early: **model your own worst case, not the enemy's mind.** If a creep survives *any* focus the defense could put on it, you don't care what they actually do. That single move turns "what will they do this tick," which is unanswerable, into "what's the worst they *could* do," which is arithmetic over the towers and defenders in range. Almost everything downstream depends on that swap.

You also need a few Screeps facts in your bones, because they shape the math:

- Distance is **Chebyshev** — 8-connected, square rings, no hex.
- **Towers** hit room-wide and *harder the closer you are* (up to ~600/tick, falling off with distance). Depth is your only lever against them, and it barely moves.
- **Ramparts** are walls the defender shoots from behind, at range up to 3.
- **Heal** lands full at range 1, a third of that at range 2–3.
- A tick resolves in **two phases**: all attacks and heals apply on the *start-of-tick* positions, then everyone moves. Damage and heal net together in the same tick — a creep doesn't die to a hit a same-tick heal would have covered.
- HP is **per-part, front to back**. The front part eats damage first, so *part order matters* (more on that at the end).

Keep only the ones your game actually has. The design generalizes; these specifics don't.

## Build the simulator before any tactics

The highest-leverage thing you can do is also the least glamorous: write a dumb, deterministic simulator *first*, and put every tactical decision *outside* it.

The simulator knows physics and nothing else. Hand it everyone's intended moves, attacks, and heals; it returns the next state. That's the whole job:

```
step(state, intents) -> nextState
```

Every actual decision — where to move, who to heal — lives in a separate function that reads the world and proposes intents:

```
policy(state) -> intents
```

The simulator only *grades*. It's a wind tunnel: the engine is the air, each tactical idea is a shape you drop in, and nothing you're testing also gets to be the thing measuring it.

This matters entirely because the physics is pure and deterministic. Determinism is what lets you run thousands of ticks a second offline, isolate one idea, and replay any battle exactly twice. Without it, none of the A/B numbers you'll want later mean anything. Build this seam first and everything after it is measurable; skip it and you're tuning by vibes.

## One classifier both halves share

Two layers do the deciding — one moves the creeps, one routes the healing — and they have to agree without reaching into each other. The thing that lets them is a single per-creep classification. For each creep, on each tile it might step to, weigh the **worst damage it could take there** against the **most healing you could actually route to it**:

```
worst = worstDamage(tile)   # towers at that range + worst defender fire reaching the tile
best  = bestHeal(tile)      # sum of routable heal (full at range 1, /3 at range 2-3)

if   hits         >= worst:  band = SAFE     # survives the worst case on its own HP
elif hits + best  >  worst:  band = RESCUE   # dies on HP alone, heal can cover the gap
else:                        band = DOOMED   # no reachable heal saves it
```

Three labels, not a number, because **survival is a gate, not a score.** The blob never trades a death for progress at any exchange rate — a single Doomed creep vetoes the entire plan. A number invites a bad trade; a gate refuses it.

One subtlety is worth stealing outright, because I got it wrong first. Don't gate on the *sum* of danger over all creeps. The creep at the very front is eating the most fire but sitting at full HP and is the last one any sane defender would shoot — summed, it trips the veto and freezes the advance for no reason. **Gate on the one creep the enemy could actually kill: the thinnest survival margin, not the largest incoming number.** "Who's taking the most fire" and "who can the enemy actually kill" are different questions, and only the second should stop the blob.

`bestHeal` being a plain sum looks wrong until you remember the focus assumption: under single-target focus, when creep *i* is the one being shot, *every* healer in range of *i* can pile onto it that tick. Overlapping coverage isn't a conflict; it's the hedge.

This classifier is the *only* channel the move layer and the heal layer talk through. Spend real effort on it. One clean shared abstraction beat every clever heuristic I tried to bolt on later.

## The objective is a ladder, not a weighted sum

Given the bands, how do you score a candidate joint move for the whole blob? Not by throwing everything into one weighted sum — survival would eventually get outbid. Score it as a **lexicographic ladder**, strict tiers compared in order:

```
1. (hard gate)  number of DOOMED creeps             -> must be zero
2.              total RESCUE shortfall (with margin) -> minimize
3. (soft)       one weighted sum: advance + attack uptime + recovery + cohesion
```

You only optimize tier 3 *among* the moves that already pass 1 and 2. Survival gates advance; it is never traded against it. The soft weights can shuffle progress against firepower against staying healthy — but they can't buy any of it with a death.

Most of the soft terms are what you'd guess (step down a distance field toward the target; keep guns on target; pull wounded creeps toward the protected center; a light nudge to keep stragglers from wandering off alone). The one worth dwelling on is the survival term, because it's where I lost the most time.

For a long while every survival check I wrote asked **"are we hurt?"** — a *stock* question, about HP in the tank right now. Against a slow trickle of tower fire the blob is never visibly hurt; it just loses a sliver more than it heals each tick until someone quietly crosses zero. The question that actually mattered is a *flow* one: **"is the bleed outrunning the heal?"** Rewriting the survival term around flow instead of stock — hold a tile only if healing can cover what you'll take *while you sit there*, not just right now — was the single biggest jump in the project. On one heavily-walled bed it took attack efficiency from **0.146 to 0.719**. If you take one tuning idea from this whole post, take that one: watch flows, not stocks.

## Choosing a joint move without paying 9^N

Now the engine room. Every creep has up to 9 options (stay, or 8 directions), so the joint move space is ~9^N — hopeless to enumerate at blob sizes. Two ideas make it tractable.

**Separability.** For a single tick the enemy roster, the distance field, and the bodies are all fixed. So almost every quantity you score — worst damage, distance progress, attack uptime — is a function of `(creep, tile)` *alone*, and you can precompute it once over the ≤9N creep-tile pairs instead of re-deriving it at every one of the 9^N leaves. The *only* genuinely joint quantity is heal routing: who can heal whom depends on where everyone ends up. Isolate that, and the rest collapses.

**A frozen oracle and a fast twin.** Write a slow, exhaustive solver that is *provably* optimal — no heuristics, ever — and freeze it as your reference. Then write the fast one you'll actually run, and hold it to a brutal contract: **bit-identical moves and scores** to the oracle. If they ever disagree, the fast one is wrong by definition. This sounds like overkill until a heuristic silently changes which tied-optimal move you return and quietly costs you a creep every few battles. The frozen twin is what catches exactly that class of bug — the kind no unit test thinks to check.

The fast path itself is a **matching backbone**:

```
1. seed everyone at STAY, compute exact heal routing
2. flatten the tiers into one cost per (creep, tile), solve a
   min-cost creep -> distinct-tile assignment      # collisions handled structurally
   recompute heal, re-match, repeat to a fixpoint
3. best-response polish over three joint move classes:
     - uniform formation step  (the whole blob shifts one direction together)
     - single relocation       (one creep to a free tile)
     - pair swap               (two creeps trade -- wounded rotates into cover)
4. danger-gated branch-and-bound on top, seeded with the polished
   move, never returns worse than its seed
```

The min-cost assignment (a Jonker–Volgenant / shortest-augmenting-path solver — the same thing `scipy`'s `linear_sum_assignment` runs) is the clever bit: **collisions stop being something you clean up afterward and become a structural constraint of the match** — two creeps simply can't be assigned the same tile. The three polish move classes exist because a packed blob has no free adjacent tiles, so single-creep moves can't find the joint motions that matter: the whole body shifting a step, or a wounded creep and a healthy one swapping places. Those are the moves that keep a blob alive, and only a joint search sees them.

## Keeping it inside a CPU budget

A blob solver that's correct but slow is useless — Screeps gives you a hard CPU allowance per tick. The good news is that the same structure that makes the solver tractable makes it cheap, if you're deliberate:

- **Precompute the separable quantities once** per tick. This is most of the work, and it's O(N) in creeps, not O(9^N).
- **Bound aggressively.** In the branch-and-bound, bound each partial assignment by placing every remaining healer at its individually-best tile — an optimistic estimate that lets you prune any subtree that can't beat the incumbent.
- **Make the matching backbone the default, not the exact search.** The exhaustive solver is your *reference*, run offline; at runtime you run matching + polish + a gated B&B, which targets well under a millisecond per creep.
- **Cap the search with a fixed total node budget, shared across the tick.** A *fixed total* (not per-creep) is what holds your time-per-creep roughly constant as the blob grows — a fragmented, all-hot roster can't stack budget and blow the tick.

The general lesson for a builder: pick a backbone whose worst case you can *cap*, and keep every objective term separable so your bounds stay valid. The moment a term couples all creeps together, your precompute and your pruning both fall apart.

## The layer that owns time

Everything above is greedy over a single tick, and that's a real limitation, not a detail. A one-tick-optimal solver *cannot choose to retreat and heal*, because retreating sacrifices this tick for a payoff several ticks out. Worse, inside a tower's flat maximum-damage band every adjacent tile reads the same lethal number — there's no gradient to follow out — so a greedy policy just stands in the fire and dies.

The fix is a small layer on top that holds a decision across many ticks no single tick could justify: a state machine with three modes, **ADVANCE / RETREAT / HOLD**. It doesn't move creeps itself. It hands the *same* solver a different goal to walk toward — the target for ADVANCE, a rally point for RETREAT, an anchor tile for HOLD — and lets the solver do the per-tick work.

Two things make it transfer instead of overfit:

- **Every trigger is relative and flow-based.** Not "retreat below 5000 HP" but "retreat when the weakest creep can no longer fund its own escape," measured against that creep's own max HP and the damage actually focusable on it. Relative thresholds carry across blob sizes, bodies, and threat strengths without retuning. (And notice it's the same stock-vs-flow lesson again: bleeding isn't a reason to retreat — walking into range always costs HP — only a bleed with the buffer already spent is.)
- **HOLD needs an anchor.** If you let HOLD just "stay," the recovery term's mild outward pull drifts the blob backward a tile a tick and HOLD quietly becomes a second retreat. Pin it to the tile where danger cleared.

Here's that machine cycling advance → retreat → hold on open ground against two towers — the same case that used to wipe the blob every single run:

<div style="margin:1.5em 0; text-align:center">
  <a href="{{ '/assets/sim/blob-retreat.html' | relative_url }}" target="_blank" rel="noopener" style="display:inline-block; padding:0.7em 1.2em; border:1px solid #888; border-radius:6px; text-decoration:none; font-weight:600">&#9654;&nbsp; Open the interactive sim (new tab)</a>
  <div style="font-size:0.85em; color:#888; margin-top:0.6em; text-align:left">Two towers, open ground, six creeps: the macro layer cycling <b>advance &rarr; retreat &rarr; hold</b>. Opens a tick-stepper in a new tab &mdash; hit Play, or step one tick at a time with the arrow keys.</div>
</div>

## The half I've skipped: the body

Everything above moves a blob. What you *spawn* is the other half of the problem, and I've barely touched it here because it deserves its own treatment. Two things to know so you don't build a blob that can't fight:

Parts die front-first, so **part order is a design axis, not cosmetics.** Put a MOVE part in front and one hit can cost you mobility; put a boosted TOUGH plate up front and it buys the whole formation time *without* costing firepower, because the ranged parts behind it stay alive. And under-built bodies don't die dramatically — they **disengage**. A blob without enough heal to cover the focus it'll eat never commits in the first place; the macro layer correctly refuses to walk it into fire. So design for *damage per tick delivered*, not for "does it survive," because a body that survives by never engaging has lost anyway. This part of my own project is still open — I'll write it up when it actually holds.

## How to keep yourself honest

The only reason any of the above is trustworthy is the measurement discipline, and it's the part I'd hand a new builder first:

- **A/B every change against a frozen baseline on the *same* seeds** — never a fresh batch of battles, or noise fakes a result for you.
- **Judge on survivors and attack efficiency**, over at least 8 seeds and 500 ticks. Tick-to-tick variance is large enough to lie to you over a short run.
- **Sweep the whole ladder of scenarios, not one.** Single-point tuning lies: constants I tuned on a fortress died on a two-tower chokepoint, and most of my "wins" evaporated the moment I swept across beds. That sweep — the one that tells you you were wrong — is most of what the lab is for.

## If you want to start

Here's the whole thing in one breath. Build the deterministic simulator before any tactics, and make every idea earn its place by beating the last one on the same battles. Find the one abstraction your layers share and pour effort into it — mine was Safe / Rescue / Doomed. Make survival a gate, never a score. Watch flows, not stocks. Keep the fast path honest with a frozen reference. Put a state machine on top for the decisions no single tick can make. And write your dead ends down, because the note on *why* an idea died is the only thing that keeps you from walking back into it.

The blob is nowhere near finished, which is most of the appeal. If any of this makes you want to build your own, that was the point.
