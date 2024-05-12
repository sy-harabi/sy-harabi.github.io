---
layout: single
title: "Getting Started with Rewriting My Bot"
toc: true
toc_stickey: true
---

## Why I Started to Rewrite

I began playing Screeps in August 2022, so it’s been about 2 years now. During this time, I've primarily focused on developing my own bot, which has been an immensely enjoyable experience. If you're interested, you can see how my bot operates on [my YouTube channel](https://www.youtube.com/@harabi-ws7sk) or check out [its GitHub repository](https://github.com/sy-harabi/screeps_harabi). However, recently, I've felt compelled to completely rewrite my bot.

The previous structures have grown overly complex, making it challenging to incorporate new features or tweak existing ones. Consequently, I've concluded that a complete overhaul of my bot is necessary.

## My Goals
I dedicated several weeks to contemplating the objectives I wish to achieve through this rewrite. The knowledgeable players in the Screeps Discord channel provided invaluable guidance; it truly is an exceptional community. Here are the outcomes of my deliberations: the goals I aim to accomplish with this rewrite.

1. Establish a structure that facilitates the addition of new features and the modification of existing ones.
2. Adopt a top-down approach: create objects that oversee each mission, handling the requisition of necessary creeps and managing their behavior. I aim to avoid allowing creeps to operate autonomously.
3. Preserve the economic functionality of my current bot with minimal alterations, as its current state boasts a robust economy.
4. Revamp the room defense and remote conflict features, as they are outdated, inefficient, and excessively CPU-intensive.
5. Refine the Quad and Duo code. Although functional, their complexity impedes scalability to larger combat scenarios. I aspire to simplify and enhance their usability.
6. Implement effective CPU management by meticulously profiling each mission.
7. Optimize spawn times by dynamically spawning creeps or repurposing idle ones.

So, let’s delve into it in the next post!
