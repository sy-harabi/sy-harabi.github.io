---
layout: single
title:  "Getting Started with Rewriting My Bot"
---

## Why I Started to Rewrite and My Goals

I first started playing Screeps in August 2022, so it’s been about 2 years since then. Up until now, I've focused on writing my own bot, which has been a fun and wonderful experience. However, I've recently felt the need to entirely rewrite my bot.

The old structures have become too complicated, making it hard to add new features or modify existing ones. Therefore, I've decided that it's time for a total rewrite of my bot!

I spent several weeks pondering the goals I want to accomplish by rewriting my bot. The great players in the Screeps Discord channel were really helpful advisors for me; it's an amazing community. Here are the results of my pondering: my goals to accomplish by rewriting my bot.

1. Create a structure that is easy to add new features to and modify existing ones.
2. Primarily use the top-down method: create objects that control each mission and have them request needed creeps and govern the creeps. Avoid letting creeps think and act on their own.
3. Maintain the economy feature of my current bot with minimal changes, as my current bot already has a good economy.
4. Rewrite the room defense and remote conflict features since they are a bit outdated, inefficient, and consume too much CPU.
5. Rewrite the Quad and Duo code. While they are functional, the code is too complex, making it difficult to expand into larger combat scenarios. I want to make them easier and more flexible to use and modify.
6. Manage CPU effectively by carefully profiling each mission.
7. Manage spawn time effectively by flexibly spawning creeps or reusing creeps that have no current task to perform.

So, let’s get into it from the next post!
