---
layout: single
title: "Journey to Solving the Traffic Management Problem"

toc: true
toc_sticky: true

comments: true
---
## What is traffic management in Screeps?
Movement is one of the significant challenges in Screeps. In fact, the official Discord has a dedicated pathfinding channel. Traffic management, a critical aspect of movement control, refers to the efficient control and movement of creeps to prevent congestion and collisions. Without proper management, creeps can get stuck or move slowly, severely hindering their efficiency.

<img src = "https://github.com/mmistakes/minimal-mistakes/assets/71678452/90a2464e-9739-49e1-8cc0-3803d767a8fd" >

## Common methods to manage traffic
The most intuitive solution is swapping and pushing. This means if a creep is blocking another creep's path, you swap their positions or push the blocking creep out of the way. While effective in many cases, this method can fail in crowded areas, creating edge cases. Advanced techniques to handle these scenarios include prioritizing creeps and recursive shoving, which involves pushing a chain of creeps until one is moved to an empty tile.
## What’s different about my solution?
I found the common methods unsatisfactory due to the numerous edge cases they created, especially in extreme situations. Each new edge case required additional handling, complicating the solution. I aimed to develop a general solution to this problem. Through algorithmic research, I discovered that the Bipartite Matching Problem (BMP) could help solve this issue. Essentially, traffic management involves assigning each creep to a specific position for the next tick. By tweaking the Ford-Fulkerson method, I successfully addressed the traffic management problem.
## Basic Idea
This solution leverages graph theory. Imagine two sets of vertices: one representing creeps and the other representing room positions. We connect these vertices with edges, indicating a creep's potential move or stay. There are two types of creeps: those intending to move to an adjacent tile and those without such an intent. Our goal is to maximize the number of fulfilled move intents.
## How the method works
We start by connecting each creep to its current position with edges. For each creep with a move intent, we search for augmenting paths in our graph. If a path increases the number of fulfilled intents, we send flow along that path. We use depth-first search (DFS) to explore these paths, scoring each path as follows: +1 if it connects a creep to its intended position, and -1 if it cancels an existing connection. After iterating through all creeps, the results indicate where each creep should be in the next tick.

**WARNING:**
I thought this method ensures that we can maximize the number of creeps reaching their intended positions while adhering to the given constraints. But it turned out that it does not. To ensure that, we have to add while loop to check it's really an optimal solution. I'm not certain that will be worth it considering it will use more CPU. So I'll leave the method unchanged for a while.

## Further Explanation of the Traffic Management Algorithm with an Example
**Note:** You can skip this part if it seems too complicated. It's an optional detailed explanation of the algorithm.

Suppose we have a situation like the picture below. Here, `A`, `B`, `C`, and `D` are creeps, and `a`, `b`, `c`, `d`, `e`, and `f` are positions.

<img src = "https://github.com/mmistakes/minimal-mistakes/assets/71678452/20899770-2bd0-4ed2-ad8d-7a787b32372a" style = "width:50%">

- Creep `B` wants to move to `a`
- Creep `C` wants to move to `b`
- Creep `D` wants to move to `a`
- Creep `A` doesn't care about its movement.

We can represent this situation with the following graph:

<img src = "https://github.com/mmistakes/minimal-mistakes/assets/71678452/6c06979a-ffe3-4e9c-bf27-36539dc554fd" style = "width:50%">

- **Blue edges** indicate current positions
- **Red edges** indicate intended positions
- **Black edges** indicate possible positions

Our task is to match each creep to a position, satisfying the following conditions:

1. Each creep must be matched to one of the current, intended, or possible positions.
2. Each creep must be matched to exactly one position.
3. No two creeps can be matched to the same position.

Our goal is to maximize the number of creeps matched to their intended positions.

Here is a desired solution, with **green edges** representing the solution:

<img src = "https://github.com/mmistakes/minimal-mistakes/assets/71678452/e8facfa1-8892-49a1-83f3-d8c7c99a1491" style = "width:50%">

### Finding the Solution

#### Step-by-Step Algorithm:

1. **Start with Creep `B`** (since `A` has no intended position).
    - Delete the blue edge between `B` and `b`, then mark it as a black edge.
      
      <img src = "https://github.com/user-attachments/assets/89f0f0db-5720-4d6b-bd02-2a2bf0292814" style = "width:50%">

    - Find a path following these rules:
        1. You must choose a red or black edge when moving from a creep to a position.
        2. You must choose a blue or green edge when moving from a position to a creep.
        3. Visit each vertex (either creep or position) only once.
        4. When choosing a red edge, add +1 to the score.
        5. When choosing a green edge, subtract -1 from the score.
        6. If you arrive at a position which don't have blue nor green edge, return the path.

3. **Finding Paths:**
    - Choose the edge from `B` to `a` and get a score of +1.
      
      <img src = "https://github.com/user-attachments/assets/276aeb22-6e7b-49fe-8c44-5dd53fd9dcf8" style = "width:50%">
   
    - Next, choose `A` and then `b`. Since `b` has no blue or green edge, return this path.
      
      <img src = "https://github.com/user-attachments/assets/333ed95c-08f4-4e6a-894d-d0eabcecd428" style = "width:50%">

    - Calculate the total score of the path. If it’s positive, apply the path to the graph; if not, ignore the path and revert the graph. The score here is +1, so we apply the path.

4. **Update the Graph:**
   - Convert the edges along the path to green(when it was red before) or blue(when it was blue or black before) edges when it's from a creep to a position. When it's from a position to a creep, convert it back to the red(when it was red before) or black(for the rest) edges accordingly.

     <img src = "https://github.com/user-attachments/assets/f62d69b2-dee6-415c-951f-35acd26b926e" style = "width:50%">
   
    - The updated graph shows that `B` is now matched to its intended position `a`.

6. **Repeat for Remaining Creeps:**
    - Now, process `C`. Start from `C` to `b`, then go to `A`.

      <img src = "https://github.com/user-attachments/assets/2fb21c16-3eb1-422f-9544-a7ebd5ecebb4" style = "width:50%">
   
    - At `A`, try different positions (`a`, `d`, `e`) until you find a successful path. In this case, `A` to `e` works.

      <img src = "https://github.com/user-attachments/assets/d399cbe6-a6fe-4e4a-a563-69fd7c5b0278" style = "width:50%">
   
    - Apply this path, making `C` happy by matching it to `b`.

      <img src = "https://github.com/user-attachments/assets/51dccb20-0635-4233-ade8-6d0feecd7343" style = "width:50%">
    
    - Process `D` next. Start from `D` to find possible paths.
    
      <img src = "https://github.com/user-attachments/assets/70584a63-6505-42e6-bcce-314f4eeec614" style = "width:50%">

    - If the path results in a negative score, revert and end the process. Here, the path has a score of -1, so we do not apply it.

### Final Solution:

<img src = "https://github.com/user-attachments/assets/09d7d5fa-ce28-4261-9123-60ca30ba8e8f" style = "width:50%">

- **Result:** Move `Creep A` to `e`, `B` to `a`, `C` to `b`, and `D` stays in place (`d`).

## Pseudocode
```
Declare movementMap
Declare visitedCreeps

Function run(room):
        For each creep in creepsInRoom:
            Match creep to its current coordinate
            If creep has intended move, add to creepsWithMovementIntent

        For each creep in creepsWithMovementIntent:
            If creep is matched to its intended coordinates, continue
            Clear visitedCreeps
            Remove creep from movementMap
            If dfs(creep) > 0, continue
            Reassign creep to its current coordinate

        For each creep in creepsInRoom:
            Move creep towards matched coordinate

Function dfs(creep, score = 0):
    Mark creep as visited
    For each possible position:
        If creep is matched to its intended coordinate, increase score
        If position is unoccupied:
            If score > 0, match creep to position
            Return score
        If position is occupied by anotherCreep and anotherCreep is not visited
            If dfs(anotherCreep, score) > 0:
                match creep to position
                return the result of dfs
    Return -Infinity


Function getPossiblePositions(creep):
    Initialize possiblePositions with current position
    If intended coordinate exists:
        add to possiblePositions
        return possiblePositions
    Add valid adjacent positions to possiblePositions
    Return possiblePositions
```
## Git repository of my solution
You can read or use my code if you want to try. My screeps traffic manager is available [here](https://github.com/sy-harabi/Screeps-Traffic-Manager)


