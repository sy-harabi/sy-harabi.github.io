---
layout: single
title: "Where Should Long-Lived Data Live? Memory, Heap, and Segments in Screeps"

toc: true
toc_sticky: true

comments: true

categories:
  - internals
tags:
  - memory
  - heap
  - segments
  - performance
---

There are three main places to store data across ticks in Screeps:

1. `Memory`
2. the JavaScript heap
3. RawMemory segments

They differ in persistence, CPU cost, available space, and how quickly the data can be accessed.

Most players use `Memory` first, since it is introduced in the Screeps tutorial. It is simple and persistent, and it works well for many kinds of bot state.

As a bot grows, however, storing everything in `Memory` becomes inefficient or inconvenient. Heap and RawMemory segments provide different tradeoffs.

# Memory

`Memory` is the standard persistent storage provided by Screeps.

```js
creep.memory.role = "miner"
Memory.targetRoom = "W10N20"
```

Data stored in `Memory` survives both ticks and global resets.

## Memory stores JSON-compatible data

`Memory` is serialized using JSON.

This means it works well with values such as:

```js
{
    role: "miner",
    sourceId: "5bbcaf...",
    lastSeen: 12345678,
    active: true
}
```

but it cannot properly preserve arbitrary JavaScript objects such as:

- `Map`
- `Set`
- functions
- class instances
- `CostMatrix`
- Screeps game objects such as `Creep`, `Room`, `Source`, or `Structure`

For Screeps game objects, store a stable identifier instead.

```js
creep.memory.sourceId = source.id
```

Then retrieve the current object when it is needed:

```js
const source = Game.getObjectById(creep.memory.sourceId)
```

For objects identified by name, such as creeps, store the name instead:

```js
const creep = Game.creeps[creepName]
```

This is important because game objects are recreated from the current game state every tick. A reference to a `Source` or `Creep` from a previous tick should not be treated as persistent state.

A useful rule is:

> Store IDs or names, not game object references.

## Memory has a recurring serialization cost

Persistent Memory is actually stored as a string.

With the standard Memory API, Screeps parses that string with `JSON.parse` when `Memory` is first accessed during a tick. The resulting object is serialized back with `JSON.stringify` at the end of the tick.

The CPU cost of this work is paid by the player's script.

Conceptually, the normal behavior is approximately:

```js
Memory = JSON.parse(RawMemory.get())

// player code

RawMemory.set(JSON.stringify(Memory))
```

This means that Memory has a recurring serialization cost.

A large value that rarely changes still contributes to the cost of parsing and serializing Memory on ticks where Memory is used.

Normal Memory is also limited to 2 MB.

## When Memory is useful

Memory is well suited to data that:

- must survive a global reset,
- should be available immediately,
- is relatively small,
- represents state that should not simply be discarded and reconstructed.

Typical examples include:

- creep roles,
- creep assignments,
- mission state,
- configuration,
- timers,
- persistent strategic decisions.

For example:

```js
creep.memory.sourceId = source.id
```

may represent an assignment rather than a cache.

Even if selecting a source again would be cheap, a global reset should not necessarily cause the creep to choose a different one.

The same applies to a strategic decision:

```js
Memory.attackTarget = "W10N20"
```

The important question is not only whether the value can be calculated again.

It is whether losing the exact state is acceptable.

# Heap

Heap is ordinary JavaScript runtime memory.

There is no Screeps API called `Heap`. In Screeps discussions, "heap" usually refers to JavaScript values that remain alive in the current runtime between calls to `loop()`.

For example:

```js
const pathCache = new Map()

module.exports.loop = function () {
    // pathCache is still available on later ticks
}
```

The important part is that `pathCache` is declared outside `loop()`.

Code at module scope is executed when the JavaScript global environment is initialized. The same `Map` can then remain available on following ticks until a global reset occurs.

Compare this with:

```js
module.exports.loop = function () {
    const pathCache = new Map()
}
```

This creates a new `Map` every tick, so it cannot be used for cross-tick storage.

Values can also be stored explicitly on `global`:

```js
global.pathCache ??= new Map()
```

Module-level variables are usually enough, but both approaches use the same runtime memory.

## Heap stores JavaScript values directly

Unlike Memory, heap data does not need to be converted to JSON.

It can contain normal JavaScript structures such as:

- `Map`
- `Set`
- typed arrays
- class instances
- functions
- `CostMatrix`
- other arbitrary JavaScript values

There is no JSON serialization or deserialization cost simply for keeping these values in heap.

This makes heap useful for data such as:

- cached paths,
- CostMatrices,
- inter-room routes,
- distance maps,
- results of expensive searches,
- decoded segment data,
- other derived runtime state.

For example:

```js
const routeCache = new Map()

function getRoute(from, to) {
    const key = `${from}:${to}`

    let route = routeCache.get(key)

    if (route === undefined) {
        route = Game.map.findRoute(from, to)
        routeCache.set(key, route)
    }

    return route
}
```

The calculated route can remain in its normal JavaScript representation and be reused on later ticks.

## Game object references should not be kept across ticks

A heap can technically contain a reference to a `Creep`, `Room`, `Source`, or another game object.

That does not make the reference suitable for cross-tick use.

Game objects represent the current tick. If a bot needs to remember one across ticks, it should keep its ID or name instead.

For example:

```js
const sourceId = source.id

// later tick
const source = Game.getObjectById(sourceId)
```

Heap can preserve arbitrary JavaScript values, but it does not change the lifetime of Screeps game objects.

## Heap does not survive a global reset

Heap belongs to the current JavaScript runtime.

When a global reset occurs, all heap data is lost.

A cache containing hundreds of routes before a reset will be empty afterward.

Code using heap must therefore be able to reconstruct the data when necessary.

This makes heap a good choice for data that:

- should be reused across ticks,
- does not need to survive a global reset,
- can be derived or loaded again.

The calculation does not have to be cheap.

A calculation may consume significant CPU and still be a good heap candidate if recalculating it once after an occasional global reset is acceptable.

A useful rule is:

> Use heap for data that should survive between ticks but does not have to survive a global reset.

## Heap requires cache management

Heap is not unlimited.

Large or continuously growing caches increase memory usage and garbage-collection pressure.

A bot should therefore remove data that is no longer useful.

A common approach is to record when a cache entry was last used:

```js
{
    value: matrix,
    lastUsed: Game.time
}
```

and periodically remove old entries.

Cached values may also become invalid even when they are still being used.

For example, a cached room `CostMatrix` may become incorrect after a structure is built or destroyed.

Cross-tick heap caches may therefore need:

- invalidation,
- expiration,
- cleanup.

# RawMemory segments

RawMemory segments provide another form of persistent storage.

They are less commonly used by newer players because their access model is more complicated than normal Memory.

Screeps provides 100 segment IDs, numbered from 0 to 99. Each segment can contain up to 100 KB, for a total capacity of 10 MB.

At most 10 segments can be active at the same time.

## Segment access is asynchronous

A segment that is not currently active cannot be read immediately.

The bot first requests it:

```js
RawMemory.setActiveSegments([0])
```

The segment becomes available on the following tick:

```js
const data = RawMemory.segments[0]
```

This activation delay is the main complication of using segments.

Writing does not require waiting an additional tick once a writable segment slot is available. The bot assigns a string during the current tick:

```js
RawMemory.segments[0] = data
```

and Screeps persists that value between ticks.

A larger segment system therefore usually needs to manage:

- which segments are currently loaded,
- which segments should be requested next,
- which segments have changed and need to be written.

## Segments store strings

RawMemory segments do not automatically contain JSON objects.

They contain strings.

This is valid:

```js
RawMemory.segments[0] = "hello"
```

JSON is simply a common format:

```js
RawMemory.segments[0] = JSON.stringify(roomIntel)
```

and later:

```js
const roomIntel = JSON.parse(RawMemory.segments[0])
```

A bot may instead use packed strings, compression, or another custom representation.

## Serialization is controlled by the bot

This is one of the main differences between normal Memory and segments.

Normal Memory is automatically parsed and serialized as part of using the standard Memory API.

Segments expose their strings directly.

If a bot stores JSON in a segment, `JSON.parse` runs only when the bot chooses to deserialize that segment. `JSON.stringify` runs only when the bot chooses to serialize data for writing.

A large persistent dataset can therefore remain stored without being parsed and serialized every tick.

For example, a bot may store intel for thousands of rooms but only activate and decode the relevant segments when that information is needed.

## When segments are useful

Segments are well suited to data that:

- must survive global resets,
- is relatively large,
- does not need to be immediately available,
- changes infrequently enough that asynchronous access is practical.

Typical examples include:

- large room-intel databases,
- base plans,
- packed planning data,
- other large persistent datasets that do not need to be available every tick.

A useful rule is:

> Use segments for large persistent data when delayed access is acceptable.

# Combining storage types

Memory, heap, and segments are not mutually exclusive.

A persistent representation may live in Memory or a segment while a decoded or derived representation remains in heap.

For example:

```text
RawMemory segment
    serialized room intel
            |
            | deserialize
            v
Heap
    decoded room intel
```

The segment survives global resets.

The heap copy avoids repeatedly decoding the same data while the current global environment remains alive.

After a global reset, the heap cache can be rebuilt from the segment.

Memory and heap can be combined in the same way:

```text
Memory
    persistent IDs and decisions
            |
            v
Heap
    derived indexes and caches
```

The same logical information may therefore exist in several representations with different lifetimes.

# Comparison

| | Memory | Heap | RawMemory segments |
| --- | --- | --- | --- |
| Survives ticks | Yes | Yes | Yes |
| Survives global reset | Yes | No | Yes |
| Immediately available | Yes | Yes | Only when active |
| Stored form | JSON-compatible data | JavaScript values | Strings |
| Automatic serialization | Yes | No | No |
| Capacity | 2 MB | Runtime heap limit | 100 × 100 KB |
| Typical use | Small persistent state | Reconstructible runtime data | Large persistent datasets |

Some common examples:

| Data | Suggested storage |
| --- | --- |
| Creep role | Memory |
| Assigned source ID | Memory |
| Current strategic target | Memory |
| Cached path | Heap |
| Room CostMatrix | Heap |
| Inter-room route | Heap |
| Decoded segment data | Heap |
| Large room-intel database | Segment |
| Base plan | Segment |
| Packed planning data | Segment |

# Choosing where data should live

A simple decision process is:

```text
Must the data survive a global reset?
        |
       No ─────────────→ Heap
        |
       Yes
        |
Does it need to be immediately available?
        |
       Yes ────────────→ Memory
        |
       No
        |
Is it large enough that keeping it in Memory is undesirable?
        |
       Yes ────────────→ RawMemory segment
        |
       No ─────────────→ Memory
```

This is only a starting point.

How frequently the data changes, how expensive it is to serialize, and how much complexity segment scheduling adds can also affect the choice.

In general:

> Use Memory for relatively small persistent state that should always be available.

> Use heap for cross-tick data that can be reconstructed after a global reset.

> Use segments for larger persistent data when asynchronous access is acceptable.

For Screeps game objects:

> Store the ID or name, not the object itself.
