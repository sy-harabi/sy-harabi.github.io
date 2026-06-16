---
layout: single
title: "Theorycrafting a Combat Blob: A Research Log"

toc: true
toc_sticky: true

comments: true
---

A **blob** is the least structured attack force in Screeps: a loose pack of combat creeps, no formation, that you shove at an enemy base and hope survives. No clever shape to hold, nothing to micro, *or so it looks*. Anyone who's tried it knows better. Keeping a blob alive on a defended room is one of the game's nastier movement problems, and I'd left it alone for a long time precisely because it looked too hard to be worth the trouble. What changed my mind was watching other players pull it off: the blobs droidFreak and iamgqr have been running lately were equal parts inspiration and a push to finally start.

Knowing how hard it is, I didn't want to wing it. So this series is less a victory lap than a build log: how I started from a testbed instead of tactics, and how a vague goal, "move the blob in," turned into a handful of precise, measurable questions. This first post is the map. It's mostly a tour of the ideas that *didn't* work, because that's where the questions got sharp.

<div style="text-align:center; margin:1.5em 0">
  <iframe width="315" height="560" src="https://www.youtube.com/embed/063zXcRSQhc" title="A blob in the field" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>
  <div style="font-size:0.85em; color:#888; margin-top:0.4em">One of mine, live on the server.</div>
</div>

## Three goals that won't sit still

Strip the blob down and there are only three things it's trying to do, and they pull against each other every tick:

- **Survive**: don't let a single creep die to focused fire.
- **Advance**: close the distance to whatever you came to kill.
- **Park**: once in range, hold still and keep as many attackers shooting as possible.

Advancing means walking into massed tower fire that only gets worse the closer you get; parking deep is where you do damage and also where you die. And the enemy is a black box: I don't get to know who they'll shoot. The first real decision of the project was to stop guessing. **I never model the enemy's mind, only my own worst case.** If a creep survives *any* focus the defense could put on it, I don't care what it actually does. That one move turns "what will they do," which is unanswerable, into "what's the worst they *could* do," which is just arithmetic over the towers and defenders in range.

That's the pattern the whole series is really about: taking a question I can't answer and beating it into one I can compute and measure.

## Physics first, policy second

Before writing a line of tactics I built a deliberately dumb simulator. It knows Screeps combat physics: how damage lands, how heal nets against it, how moves and fatigue resolve, and nothing else. Hand it everyone's intended moves, attacks, and heals for a tick and it returns the next state. That's the entire job.

Every actual decision (where to move, who to heal) lives *outside* it, as a function that reads the world and proposes intents. The simulator only grades. It's a wind tunnel: the engine is the air, each tactical idea is a shape I drop in and measure, and nothing I'm testing also gets to be the thing doing the measuring.

This is the highest-leverage decision in the project, entirely because the physics is pure and deterministic. Determinism is what lets me run thousands of ticks a second offline, isolate one idea, and replay any battle exactly the same way twice, which is the only reason the A/B numbers further down mean anything. The search layer that sits on this seam (how you pick a joint move for the whole blob each tick without it costing a fortune in CPU) is the most involved engineering in the project, and it gets its own post.

## One classifier, shared by both halves

Two layers do the deciding: one moves the creeps, one routes the healing. They have to agree without reaching into each other's internals, and the thing that lets them is a single classification. For each creep, on each tile it might step to, I weigh the **worst damage it could take there** against the **most healing I could actually route to it**, and label the result:

- **Safe**: survives the worst case on its own HP.
- **Rescue**: dies on HP alone, but heal can cover the gap.
- **Doomed**: no reachable amount of heal saves it.

The point of three labels instead of a number is that **survival is a gate, not a score.** The blob never trades a death for progress at any exchange rate; one Doomed creep vetoes the entire plan.

My first version of that veto was wrong in a way worth keeping. It summed danger over *every* creep at once. So the creep at the very front, eating the most fire but sitting at full HP and the last one any sane defender would shoot, dumped a huge number into the sum and tripped the veto, freezing the advance for nothing. The fix was to gate on the one creep the defense would actually kill: the thinnest survival margin, not the largest incoming number. "Who's taking the most fire" and "who can the enemy actually kill" are different questions, and only the second should be allowed to stop the blob. The classifier carries more than this one fix, and I'll give it its own write-up.

## Stock vs. flow

For a while the blob handled big fortresses and then died, embarrassingly, to the cheapest defense on the board: two towers on a chokepoint. The expensive base was the easy one, and the cheap one kept killing it. I stared at that backwards result for a long time.

The bug wasn't in the code, it was in the question. Every survival check I'd written asked **"are we hurt?"** That's a *stock* question, about how much HP is in the tank right now. Against a slow trickle of tower fire the blob is never visibly hurt; it just loses a sliver more than it heals each tick until somebody quietly crosses zero. The question that actually mattered was a *flow* one: **"is the bleed outrunning the heal?"**

Rewriting survival around flow instead of stock was the turning point of the project. On a heavily-walled fortress bed it pulled attack efficiency from **0.146 to 0.719**, from flinching at the wall to locking onto the target, and the two-tower case that started the whole thing got its fix one layer up (next section). Almost everything good that came later is some version of this same swap.

## Knowing when to leave

A blob that never retreats is just a slower way to lose. But retreating is something the move layer can't do on its own, and the reason is myopia: the solver only ever optimizes the next tick. Close to a base that's fatal. Inside a tower's flat max-damage band every adjacent tile reads the same maximum hit, so a one-tick-greedy policy sees no gradient to follow out; every direction looks equally lethal, it never commits to the run of bad tiles a real escape costs, and it just stands in the fire and dies.

That's the job of the next layer up: a small state machine, **ADVANCE, RETREAT, HOLD**, that holds a decision across many ticks no single tick could justify. RETREAT hands the solver a goal to walk toward while the local field is flat, and a flow test decides when to commit to it, the moment the bleed starts outrunning the heal. (The old trigger was stock-based and fired far too late, around 12% HP, while HOLD kept re-parking the blob on the next full-HP creep and feeding one death per cycle.) On the two-tower chokepoint that used to wipe every run, this took the blob from **0 of 8 survivors to 7 of 8 clean**, though only once it also stopped scattering on the way out, which is the next section's problem.

<div style="margin:1.5em 0; text-align:center">
  <a href="{{ '/assets/sim/blob-retreat.html' | relative_url }}" target="_blank" rel="noopener" style="display:inline-block; padding:0.7em 1.2em; border:1px solid #888; border-radius:6px; text-decoration:none; font-weight:600">&#9654;&nbsp; Open the interactive sim (new tab)</a>
  <div style="font-size:0.85em; color:#888; margin-top:0.6em; text-align:left">Two towers, open ground, six creeps: the macro layer cycling <b>advance &rarr; retreat &rarr; hold</b> and back. It opens a tick-stepper in a new tab &mdash; hit Play, or step one tick at a time with the arrow keys. (The 0/8 &rarr; 7/8 figures above come from the tighter two-room chokepoint bed; this is the same machinery on open terrain, where it reads more clearly.)</div>
</div>

## Holding together without clumping to death

Creeps that drift get picked off alone, so I added a gentle pull toward the pack's center, just enough to keep a straggler from wandering off the edge and dying by itself.

This looked like a genuine no-win trade-off, and I believed it for a while. The pull's worth turns out to depend entirely on the terrain. On broken, walled maps, where a blob can get cut into pieces it can't reunite, holding together is the only thing keeping it from being divided and killed a creep at a time. On open ground there's nothing to divide it: every creep can reach its own best tile, and cohesion does nothing but drag each one off that optimum. Opposite preferences, no single setting serving both, and I almost filed it as a law of the problem.

The conflict came down to one thing: the pull was a single linear knob. One strength, applied to every creep, so anything strong enough to reel in a separating straggler tugged a settled creep off its optimum just as hard, and no value served both terrains at once. The fix wasn't a compromise number, it was a different shape. I changed the pull from linear to quadratic, the distance to the pack's center times the distance to where that center is heading, which sits near zero inside the swarm and climbs steeply only at the edge. Now it grips only the creeps actually drifting off and leaves the packed middle alone, which is what each terrain wanted all along.

(I'd tried cleverer pulls first. One tracked each creep's own heading instead of the group's; another enforced spacing with no sense of the pack's shape. Both failed for the same reason: neither carried the only signal that mattered, how far *this* creep is from everyone else. A post of its own.)

So cohesion kept its place, not as the "hold formation" rule I first imagined but as a light nudge that grips only the creeps slipping off the edge.

## The body is the other half, and the half I'm on now

I spent most of this obsessing over movement, but what you *spawn* is the other half of the problem, and it's where the work actually is right now. Part order is a live design axis: a TOUGH plate up front, boosted hard enough to buy the formation time without costing firepower, and the heal-to-ranged split set against the focus you expect to eat. I'm trying to fold all of it into a closed-form `chooseBody`: a body straight from an energy budget and a boost tier, no simulation, so the live bot can call it inline.

It is not settled. The towers-only case works; against real defenders the calibration is still open, and I've already thrown out one version I was sure was finished. This is the current frontier of the project, not a closed box. It gets written up when it actually holds, not a post sooner.

## How I keep myself honest

The only reason any of the above is trustworthy is the measurement discipline, and it's the part I'd hand a new builder first. Every change is A/B'd against a frozen baseline on the *same* seeds, never a fresh batch of battles. I judge on survivors and attack efficiency, over at least 8 seeds and 500 ticks, because tick-to-tick noise is large enough to fake a result on its own. And I sweep the whole ladder of scenarios, not one, because **single-point tuning lies**: constants I tuned on a fortress died on a chokepoint, and most of my "wins" evaporated the moment I swept across beds. That sweep, the one that tells you you were wrong, is most of what the lab is for.

## If you want to build your own

The blob is nowhere near finished, which is most of the appeal. If this makes you want to build one, here's roughly what I'd tell you.

Build the simulator before any tactics, and make every idea earn its place by beating the last one on the same battles. Spend real effort on the one abstraction your layers share. Mine was Safe / Rescue / Doomed, and one clean concept that lets the layers cooperate beat any number of clever heuristics. Watch flows, not stocks: "are we losing ground faster than we can recover it" is the better question almost everywhere I asked it. When two goals look flatly opposed and no coefficient reconciles them, suspect the quantity before the number: sometimes nothing fits because the term is the wrong shape, not the wrong size. And write the dead ends down: half of this post is failed ideas, and the note on *why* each one died is the only thing that keeps you from walking back into it.

None of the answers were the satisfying part, though. The satisfying part was almost always the sweep that proved me wrong and pointed at where to look next.
