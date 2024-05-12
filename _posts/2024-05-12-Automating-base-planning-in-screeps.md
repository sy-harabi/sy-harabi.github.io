---
layout: single
title: "Automating Base Planning in Screeps – A Step-by-Step Guide"

toc: true
toc_sticky: true

comments: true
---
## Why Base Planning?

Planning the layout of your base in Screeps might seem unusual at first, but it’s a crucial step in automating your bot. It’s also one of the most challenging tasks to tackle manually. While you don't necessarily need to fully understand the algorithms involved, you should at least know how to utilize them effectively. So, here’s a step-by-step guide on how I automate my base planning. Let’s dive into it.

## The Process

1. **Utilize Distance Transform**: Start by employing the distance transform algorithm to identify open spaces in the room. Typically, my bot requires a 5x5 square to begin, so I target positions with a distance of 3 or larger. Some bots, especially those employing bunker-style base planners, may require squares larger than 10x10. Plan ahead for the desired size of your base. The distance transform code is available [here](https://github.com/sy-harabi/harabiBot_2024/blob/main/src/util_algorithm.js#L14).
![1  distance transform](https://github.com/sy-harabi/harabiBot_2024/assets/71678452/659081f1-20cb-472c-9e4c-d96141550810)
3. **Select Starting Position**: Choose an open area close to both the controller and energy sources. Proximity to the controller is critical, especially if significant energy upgrades are anticipated. Distance to energy sources is less crucial since transportation from source to storage is capped at 10 energy per tick.
![2  start position](https://github.com/sy-harabi/harabiBot_2024/assets/71678452/88887a93-32a6-439e-bf0c-2d905f189cd2)
4. **Place Core Structures**: Begin by placing core structures such as spawn, storage, and terminal. I surround the starting position with core structures using a 5x5 stamp for efficiency. Some bots utilize a stamp called "fast filler" to quickly fill extensions. The choice of design is up to you!
![3  core structures](https://github.com/sy-harabi/harabiBot_2024/assets/71678452/0d646920-4898-4b41-a2c3-2a7274ba4f7d)
5. **Designate Controller Upgrade Area**: Allocate a 3x3 area near the core structures for upgrading the controller.
![4  upgrade area](https://github.com/sy-harabi/harabiBot_2024/assets/71678452/46b15b7e-15b7-4461-b7c6-337531dd7dde)
6. **Implement Floodfill Algorithm**: Utilize the floodfill algorithm to categorize tiles by their accessibility. Lower numbers indicate proximity to core structures. The floodfill code is available [here](https://github.com/sy-harabi/harabiBot_2024/blob/main/src/util_algorithm.js#L100).
![5  flood fill](https://github.com/sy-harabi/harabiBot_2024/assets/71678452/62e9203a-a4c2-4d3c-ab10-ae1d0848e3d4)
7. **Allocate Tile Usage**: Assign tiles for essential structures such as extensions, labs, towers, factory, nuker, and observer. I personally prefer a grid layout similar to the "commie bot."
![6  grid](https://github.com/sy-harabi/harabiBot_2024/assets/71678452/bc52073f-05f0-45dc-9b08-e64e66a1ee4e)
8. **Position Labs**: Identify 10 suitable tiles meeting specific criteria, ensuring that there are two source labs within range 2 of every other lab.
![7  labs](https://github.com/sy-harabi/harabiBot_2024/assets/71678452/94ec36fb-5979-4c72-be14-acbaaea6d48d)
9. **Establish Infrastructure**: Lay down roads from storage to energy sources and minerals. Place a container and a link at each source, and a container and an extractor at the mineral.
![8  roads to resources](https://github.com/sy-harabi/harabiBot_2024/assets/71678452/56af9375-b8f0-41c5-bd7e-b67c9945e50c)
10. **Optimize Ramparts**: Utilize the minimum cut algorithm to determine optimal rampart positions, then connect them with roads. At this stage, I made controller upgrade area to be closer to controller. The minimum cut code is available [here](https://github.com/sy-harabi/harabiBot_2024/blob/main/src/util_algorithm_mincut.js).
![10  roads to ramparts](https://github.com/sy-harabi/harabiBot_2024/assets/71678452/75b5bc4d-7165-474b-a9bc-f17e2b8d9efd)
11. **Tower Placement**: Position towers to maximize their coverage. Personally, I prioritize maximizing the minimum damage of towers over base boundaries.
![11  towers](https://github.com/sy-harabi/harabiBot_2024/assets/71678452/9a591d7c-ca7a-4542-86eb-b641a4514850)
12. **Place Additional Structures**: Situate the factory, nuker, observer, and extensions. Placement for the observer is less critical, so anywhere works. At this stage, I changed lab position a bit and marked them as green square.
![12  nuker, factory, observer, extensions](https://github.com/sy-harabi/harabiBot_2024/assets/71678452/05b7bb2e-718b-4051-bdab-1bccebf58200)

## Examples and Visuals
Here are some examples created by my bot. The upgrade area is marked with a yellow square, labs with a green square, and towers with a red square.
![13  example(1)](https://github.com/sy-harabi/harabiBot_2024/assets/71678452/5b071085-2947-46d2-a1a5-715b88aa10d5)
![14  example(2)](https://github.com/sy-harabi/harabiBot_2024/assets/71678452/7fcb979e-2b26-4315-ae4d-1ee9efaa9d66)
![15  example(3)](https://github.com/sy-harabi/harabiBot_2024/assets/71678452/ec006a28-dbdb-4295-9c3b-0a66ff006a24)
![16  example(4)](https://github.com/sy-harabi/harabiBot_2024/assets/71678452/14fb97be-2b4d-4980-bbd1-c1b3879e4cbc)
![17  example(5)](https://github.com/sy-harabi/harabiBot_2024/assets/71678452/70c182d3-6148-4c91-aac0-debb4b9aac7a)
Automating base planning streamlines your development process and lays a solid foundation for your Screeps bot to thrive.
