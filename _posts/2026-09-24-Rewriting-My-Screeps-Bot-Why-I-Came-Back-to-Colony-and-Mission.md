---
layout: single
title: "Rewriting My Screeps Bot – Why I Came Back to Colony + Mission"
toc: true
toc_sticky: true
comments: true

categories:
  - bot-design
tags:
  - architecture
  - rewrite
---

# Rewriting My Screeps Bot – Why I Came Back to Colony + Mission

Recently, I started rewriting my Screeps bot from scratch.

My existing bot already works quite well, so I did not really need a rewrite. I had three main reasons:

1. I wanted to learn TypeScript.
2. I wanted to improve some parts of my old bot.
3. I thought writing a new bot would be fun.

The third reason was probably the biggest one.

Since I was starting from scratch, I also wanted to try a different architecture instead of just rewriting the old JavaScript code in TypeScript.

My first idea was an **Operation-based architecture**.

I imagined the whole bot as something similar to an HTN tree, with an `EmpireOperation` at the root. Almost everything the bot did would be represented somewhere in this tree.

I liked the idea of describing a complex Screeps bot with one common structure.

While implementing the economy and logistics, however, I started to find cases that did not fit this structure very well.

I eventually changed it back to something similar to my old bot: **Colonies + Missions**.

This post explains why.

---

## The Initial Idea: One Operation Tree

The basic idea was to represent almost everything the bot does as an Operation.

At the top would be an `EmpireOperation`.

It could create child Operations, and those Operations could create more child Operations.

For example:

```text
EmpireOperation
├── ColonyOperation
│   ├── MiningOperation
│   ├── BuildOperation
│   └── ...
├── TotalWarOperation
│   ├── SiegeOperation
│   │   ├── SquadOperation
│   │   └── SquadOperation
│   └── ...
└── ...
```

This was loosely based on an HTN-style hierarchy.

A large problem could be divided into smaller problems, and those could be divided again until the leaves were close to actual creep actions.

Each Operation would also use the same basic lifecycle:

```text
Plan
  ↓
Allocate
  ↓
Execute
```

During `plan`, Operations would inspect the current state and make decisions.

During `allocate`, shared resources such as creeps and spawn capacity would be assigned.

During `execute`, they would perform the actual actions.

---

## Why I Thought This Would Work

Screeps bots become complicated very quickly.

Even the economy requires mining, spawning, hauling, upgrading, building, remote mining, defense, and several other systems.

Combat adds another large set of decisions on top of that.

I wanted to divide this complexity into smaller levels.

For example, combat could look roughly like this:

```text
Empire
  ↓
Total War
  ↓
Siege
  ↓
Squad
  ↓
Creep actions
```

The higher level would make broader decisions.

A Total War Operation might decide which rooms should be attacked.

A Siege Operation would receive one of those targets and decide how to attack it.

A Squad Operation would handle the actual movement and combat of a squad.

I expected the same idea to work for the economy.

A Colony Operation could make colony-level decisions, while its child Operations handled more detailed parts such as mining or building.

This also gave me a natural place to separate **decision making from actions**.

`plan` would decide what should happen.

`allocate` would distribute shared resources.

`execute` would perform the game actions after those decisions were made.

I also liked having one common structure.

Economy, expansion, scouting, and combat could all exist in the same tree and follow the same lifecycle.

At this point, the structure looked simple and consistent.

So I started implementing it.

---

## Logistics Was Where It Started to Break

The first clear problem appeared while implementing logistics.

The logistics system needs several steps.

First, different systems determine their current supply and demand.

For example:

- miners or containers can provide energy,
- builders and upgraders can request energy,
- spawns and extensions can request energy,
- haulers can already be carrying resources.

Then all of this information has to be collected.

After that, the logistics matcher assigns suppliers to requests.

Finally, the haulers execute those assignments.

Roughly:

```text
create supply / demand
        ↓
collect everything
        ↓
match suppliers and requests
        ↓
run haulers
```

The problem was deciding where these steps belonged in:

```text
Plan → Allocate → Execute
```

In the code I was writing, some logistics information naturally appeared while running the actual subsystem.

For example, builder and upgrader logic could create energy requests.

Hauler logic also included more than just `move`, `withdraw`, and `transfer`. It had logic for deciding its current state, checking its existing assignment, and participating in the logistics system.

So if all Operations finished `plan` first and logistics matching happened during `allocate`, some of the information needed by the matcher did not exist yet.

I could move more of this logic into `plan`.

But then `plan` would need to run parts of the builder, upgrader, and hauler logic just to prepare logistics information.

The role logic would become split between `plan` and `execute`.

The other option was to register logistics information during `execute`.

But then execution order became important.

If one Operation ran its haulers before another Operation registered its energy requests, the result would be different.

So logistics needed something like:

```text
run some systems
        ↓
collect their logistics information
        ↓
resolve logistics
        ↓
continue execution
```

That did not fit the original three phases very well.

---

## I Tried Adding Another Phase

One idea was to add another phase:

```text
Plan
  ↓
Allocate
  ↓
Execute
  ↓
Finalize
```

This would give the parent or other systems another chance to process information produced by child Operations.

It could work.

But then I had to decide what exactly belonged in each phase.

Should logistics matching happen in `allocate` or `finalize`?

If it happens in `finalize`, when should the haulers actually run?

If some actions happen before logistics and some happen after logistics, `execute` is no longer one clear phase anyway.

The lifecycle was becoming more complicated to preserve the common lifecycle itself.

For the colony economy, a direct execution order was easier.

Something like:

```text
runHarvest()
runUpgrade()
runBuild()
...
runLogistics()
```

The exact order can be changed when necessary.

More importantly, if one system has to prepare information before another system runs, that relationship is directly visible in the colony code.

I did not need to map every step to a general Operation phase.

---

## One Structure Was Too Restrictive

At this point, I still liked two parts of the Operation idea.

Hierarchical decomposition is useful.

Separating decisions from actions is also useful.

The problem was trying to use the same hierarchy and the same lifecycle for the whole bot.

Different parts of Screeps have different control flows.

A strategic combat problem can naturally be divided from high-level decisions into smaller objectives.

A colony economy is different.

Several systems share state, and their order inside one tick can matter.

Some decisions are also made at different times. There is not always a clean sequence where every system can first finish all planning, then all allocation happens, and then all execution starts.

I could keep adding phases and rules to the Operation framework.

But then the common structure was no longer making the bot easier to understand.

So I stopped trying to make the whole bot one Operation tree.

---

## Back to Colony + Mission

My old bot already had a similar separation.

It had room-level management for the economy and a separate Mission system for larger objectives.

For the rewrite, I returned to the same basic idea.

### Colony

A Colony manages the continuous economy supported by one owned room.

For example:

```text
Colony
├── Mining
│   ├── local sources
│   └── remote sources
├── Spawning
├── Logistics
├── Upgrading
├── Construction
└── Local defense
```

Mining belongs to the Colony even when the source is in another room.

Remote sources still use that colony's spawn capacity.

Their haulers are part of the same hauling system.

Their income is used by the same colony.

So I treat local and remote mining as one colony economy.

The systems inside a Colony do not need to follow one common lifecycle.

The Colony can run them in the order required by the economy.

If one system needs to register something before logistics runs, I can just put it before logistics.

### Mission

A Mission represents a separate objective with its own state and lifecycle.

A siege is a good example.

The decision to attack a room should happen above the Siege Mission.

For example, an empire-level combat system may decide to attack `W1N1` and create a Siege Mission for that target.

The Siege Mission then handles the actual objective:

```text
target: W1N1
    ↓
decide attack method
    ↓
request creeps from colonies
    ↓
spawn and boost squads
    ↓
travel
    ↓
attack
    ↓
adjust if necessary
    ↓
finish or abort
```

The Mission may use resources from several colonies, but it does not belong to any one colony economy.

It also has a clear start and end.

This makes it a useful boundary for objectives such as a siege.

A Mission can still use hierarchical decomposition internally when that is useful.

For example, a Siege Mission may coordinate several squads, while each squad handles its own movement and combat.

I am not removing the hierarchical idea.

I am just using it where it fits.

---

## The Current Architecture

The exact details are still changing, but the current structure is roughly:

```text
Empire
│
├── Empire-level managers
│   ├── Resource
│   ├── Combat
│   ├── CPU
│   └── ...
│
├── Colonies
│   ├── Mining
│   ├── Spawning
│   ├── Logistics
│   ├── Upgrading
│   ├── Construction
│   └── Local defense
│
└── Missions
    ├── Siege
    ├── Claim
    └── ...
```

There are also systems shared by these parts:

```text
Intel
Navigator
Traffic Manager
Runtime Cache
Base Planner
...
```

Colonies handle continuous colony economies.

Empire-level managers make decisions that need an empire-wide view.

Missions handle separate objectives.

The shared systems are just used where they are needed.

This is quite close to the general structure of my old bot.

---

## Where the Rewrite Is Now

So the architecture ended up closer to my old bot than I expected.

Most of the changes are now inside that structure rather than replacing the structure itself.

The new bot is written in TypeScript.

The base planner has been rewritten and improved.

Movement and traffic management have been separated and rewritten.

The colony economy, logistics, and spawning systems are being rebuilt with clearer responsibilities.

I am also paying more attention to CPU usage, heap allocation, and runtime caches than I did when writing the old bot.

Scouting and intel are being rewritten as well, with some changes to the actual behavior instead of only code cleanup.

There is still a lot left to implement, especially combat.

For now, Colony + Mission is the structure I am using while continuing the rewrite.
