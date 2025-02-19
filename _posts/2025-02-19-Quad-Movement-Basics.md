---
layout: single
title: "Quad Movement Basics"

toc: true
toc_sticky: true

comments: true
---

## Quad Movement basics

![Desktop 2025 02 19 - 14 11 26 02 DVR_5](https://github.com/user-attachments/assets/73121e2f-593e-4146-b4f0-d7e3f05d3362)

In Screeps, quads play a crucial role, especially when attacking rooms with high RCL. Using only duos can make it really challenging to take on an RCL8 room. Today, I’ll introduce the basics of quad movement. 

The rough idea is simple:

1. Get four creeps.
2. Find a wide enough area for them to be packed as a square.
3. Pack them into that area.
4. Adjust the cost matrix for quads.
5. Make sure the quad is packed before finding the path.
6. Find a path to the goal.
7. Make the quad follow the path.

The most tricky part is adjusting the cost matrix, but it’s not as hard as it sounds. Let’s dive into it!

---

### Step 1: Spawn Four Creeps

First, spawn four creeps.

```javascript
Game.spawns["Spawn1"].spawnCreep([MOVE], "Quad1")
Game.spawns["Spawn1"].spawnCreep([MOVE], "Quad2")
Game.spawns["Spawn1"].spawnCreep([MOVE], "Quad3")
Game.spawns["Spawn1"].spawnCreep([MOVE], "Quad4")
```

![3  spawn 4 creeps](https://github.com/user-attachments/assets/e1dc3ea0-e6dc-4c2f-afe1-6a30425e4db5)

Now you have four creeps!

---

### Step 2: Find a Square Area to Pack Creeps

Next, you need to find some square area to pack those creeps. This can be done by searching RoomPosition objects near your creeps to find a position A such that:

- A
- Right of A
- Bottom of A
- Bottom-right of A

...are all empty.

![4  find area to form](https://github.com/user-attachments/assets/cfec820d-8b30-4be1-8db4-936b34b41883)

#### Important:

You should remember that area until the creeps are all packed together. Searching for a new area every tick could make your quad fall into an infinite loop.

Once you find the area, assign each creep to a specific position and make them go there.

```javascript
// Example: Assign formation positions relative to the top-left corner
creep1.moveTo(topLeftPos)
creep2.moveTo(new RoomPosition(topLeftPos.x + 1, topLeftPos.y, roomName))
creep3.moveTo(new RoomPosition(topLeftPos.x, topLeftPos.y + 1, roomName))
creep4.moveTo(new RoomPosition(topLeftPos.x + 1, topLeftPos.y + 1, roomName))
```

![5  gather creeps](https://github.com/user-attachments/assets/2110fd30-6346-4f4f-befc-a068606eb9ff)

Now you have four creeps perfectly packed into a square!

To ensure your creeps are packed, you can create a helper function:

```javascript
function isQuadPacked(creeps) {
  if (creeps.length !== 4) return false
  for (let i = 0; i < creeps.length; i++) {
    for (let j = i + 1; j < creeps.length; j++) {
      if (!creeps[i].pos.isNearTo(creeps[j].pos)) return false
    }
  }
  return true
}
```

This function checks that every creep is adjacent to every other creep.

---

### Step 3: Adjust the Cost Matrix

Now, it’s time to adjust the cost matrix for quad movement.

#### Concept:

- Assume the top-left creep in the formation is your "leader."
- Create a modified cost matrix that accounts for the fact that the quad occupies a 2x2 area, not a single tile.

---

![Adjust costs](https://github.com/user-attachments/assets/06278438-3933-433d-9819-ed91c227b95a)

See how the cost in the red circle has changed!

#### Example: Cost Matrix Transformation

Here’s how you can transform your existing room cost matrix for quad movement:

```javascript
function transformCosts(costs, roomName, swampCost = 5, plainCost = 1) {
  const terrain = Game.map.getRoomTerrain(roomName)
  const result = new PathFinder.CostMatrix()
  const formationVectors = [
    { x: 0, y: 0 }, // top-left
    { x: 1, y: 0 }, // top-right
    { x: 0, y: 1 }, // bottom-left
    { x: 1, y: 1 }, // bottom-right
  ]

  for (let x = 0; x < 50; x++) {
    for (let y = 0; y < 50; y++) {
      let cost = undefined

      for (let vector of formationVectors) {
        const newX = x + vector.x
        const newY = y + vector.y

        if (newX < 0 || newX > 49 || newY < 0 || newY > 49) {
          continue
        }

        let newCost = costs.get(newX, newY)

        if (newCost === 0) {
          const terrainMask = terrain.get(newX, newY)
          if (terrainMask === TERRAIN_MASK_WALL) {
            newCost = 255
          } else if (terrainMask === TERRAIN_MASK_SWAMP) {
            newCost = swampCost
          } else {
            newCost = plainCost
          }
        }

        if (cost === undefined) {
          cost = newCost
        } else {
          cost = Math.max(cost, newCost)
        }
      }

      result.set(x, y, cost)
    }
  }
  return result
}
```

---

### Step 4: Pathfinding with Quad Formation

Use the transformed cost matrix in your PathFinder.search call, and make sure to use the top-left creep's position as the startPos.

```javascript
const costMatrix = transformCosts(existingCostMatrix, roomName)
const path = PathFinder.search(
  topLeftPos,
  { pos: targetPos, range: 1 },
  {
    plainCost: 1,
    swampCost: 5,
    roomCallback: () => costMatrix,
  },
)
```

---

### Step 5: Move the Quad

![Desktop 2025 02 19 - 14 11 26 02 DVR_4](https://github.com/user-attachments/assets/56d97fd6-788d-416b-bf2e-382580b7c957)

Once you have a path, make each creep follow the path, staying in formation. A simple way is to find the direction from the path and move all creeps in that direction:


```javascript
const direction = topLeftPos.getDirectionTo(path[0])
creeps.forEach((creep) => creep.move(direction))
```

---

### Pseudocode for Overall Quad Movement

```pseudocode
// 1. Spawn four creeps, each with one MOVE part.
spawnCreep([MOVE], 'Quad1')
spawnCreep([MOVE], 'Quad2')
spawnCreep([MOVE], 'Quad3')
spawnCreep([MOVE], 'Quad4')

// 2. Check if the quad is properly packed.
if isQuadPacked(creeps) is false:
    // 2a. Find a square area (topLeftPos) where a 2x2 area is empty.
    topLeftPos = findEmptySquareAreaAroundCreeps(creeps)

    // 2b. Move each creep to its designated position to form a quad.
    creep1.moveTo(topLeftPos)
    creep2.moveTo(position at (topLeftPos.x + 1, topLeftPos.y))
    creep3.moveTo(position at (topLeftPos.x, topLeftPos.y + 1))
    creep4.moveTo(position at (topLeftPos.x + 1, topLeftPos.y + 1))

// 3. Transform the room's cost matrix to account for the quad formation.
transformedCosts = transformCosts(existingCostMatrix, roomName, swampCost=5, plainCost=1)

// 4. Use the top-left position (from the quad formation) as the starting point and search for a path to the target.
path = PathFinder.search(topLeftPos, targetPos, { costMatrix: transformedCosts })

// 5. Determine movement direction from the path.
direction = getDirectionFrom(topLeftPos to next step in path)

// 6. Move all creeps in the determined direction.
for each creep in creeps:
    creep.move(direction)

// 7. Continue pathfinding and moving until the target is reached.
```

---

### Challenges to Improve Quad Movement

Now that you've mastered the basics of quad movement, it's time to tackle some more advanced challenges. Try these out and feel free to reach out if you encounter any issues or want to share your solutions! You can ping me (Harabi) on the official Screeps Discord.

- Path caching to avoid redundant calculations.
- More efficient methods for packing creeps into formation.
- Avoiding collisions with enemy creeps.
- Temporarily breaking formation to slip through narrow passages.
- Inter-room traveling for long-range attacks or harvesting.
- Room edge handling to prevent getting stuck at exits.

---

Happy Screeping!
