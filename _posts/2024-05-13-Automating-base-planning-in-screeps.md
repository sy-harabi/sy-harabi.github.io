---
layout: single
title: "Automating Base Planning in Screeps – A Step-by-Step Guide"

toc: true
toc_sticky: true

comments: true
---
## Why Base Planning?

Planning the layout of your base in Screeps might seem unusual at first, but it’s a crucial step in automating your bot. It’s also one of the most challenging tasks to tackle on your oen from scratch. While you don't necessarily need to fully understand the algorithms involved, you should at least know how to utilize them effectively. So, here’s a step-by-step guide on how I automate my base planning. Let’s dive into it.

## The Process

1. **Utilize Distance Transform**: Start by employing the distance transform algorithm to identify open spaces in the room. Typically, my bot requires a 5x5 square to begin, so I target positions with a distance of 3 or larger. Some bots, especially those employing bunker-style base planners, may require squares larger than 10x10. Plan ahead for the desired size of your base. The distance transform code is available [here](https://github.com/sy-harabi/harabiBot_2024/blob/main/src/util_algorithm.js#L14).
![1  distance transform](https://github.com/user-attachments/assets/fb8c5f03-9e0a-4579-82ff-a21a124a5380)
3. **Select Starting Position**: Choose an open area you think the best for your base. I prefer close to both the controller and energy sources, especially proximity to the controller. But remember, everything is up to you!
![2  start position](https://github.com/user-attachments/assets/88ee32ea-99ea-445b-ba3f-bbf76c26d37c)
4. **Place Core Structures**: Begin by placing core structures such as spawn, storage, and terminal. I surround the starting position with core structures using a 5x5 stamp for efficiency. Some bots utilize a stamp called "fast filler" to quickly fill extensions. The choice of design is up to you!
![3  core structures](https://github.com/user-attachments/assets/423a1593-08a4-4552-8b50-ee5cc6055e82)
5. **Designate Controller Upgrade Area**: Allocate a 3x3 area near the core structures for upgrading the controller. I made a mistake in the screenshot below. I didn't considered properly that every position should be within range 3 to the controller. Don't make a mistake like me!
![4  upgrade area](https://github.com/user-attachments/assets/7b300577-b6a7-4a92-bfe4-28af168811cb)
6. **Implement Floodfill Algorithm**: Utilize the floodfill algorithm to categorize tiles by their accessibility. Lower numbers indicate proximity to core structures. The floodfill code is available [here](https://github.com/sy-harabi/harabiBot_2024/blob/main/src/util_algorithm.js#L100).
![5  flood fill](https://github.com/user-attachments/assets/93ff0067-7c19-40c4-9835-65a4d8caf371)
7. **Allocate Tile Usage**: Assign tiles for essential structures such as extensions, labs, towers, factory, nuker, and observer. I personally prefer a grid layout similar to the "commie bot."
![6  grid](https://github.com/user-attachments/assets/11142e54-853e-458d-9040-a906dcb6e009)
8. **Position Labs**: Identify 10 suitable tiles meeting specific criteria, ensuring that there are two source labs within range 2 of every other lab. If it seems too complicated, it's fine to use stamp for labs
![7  labs](https://github.com/user-attachments/assets/553cfa99-8983-415a-a137-63b9ec336b82)
9. **Establish Infrastructure**: Lay down roads from storage to energy sources and minerals. Place a container and a link at each source, and a container and an extractor at the mineral.
![8  roads to resources](https://github.com/user-attachments/assets/eba74c08-c41d-4c7d-8f05-52633f11b3ba)
10. **Optimize Ramparts**: Utilize the minimum cut algorithm to determine optimal rampart positions, then connect them with roads. At this stage, I noticed my mistake I mentioned earlier and made the controller upgrade area to be closer to controller. The minimum cut code is available [here](https://github.com/sy-harabi/harabiBot_2024/blob/main/src/util_algorithm_mincut.js).
![10  roads to ramparts](https://github.com/user-attachments/assets/1114dd73-d54a-4f31-9ebc-da26b38f85ee)
12. **Tower Placement**: Position towers to maximize their coverage. Personally, I prioritize maximizing the minimum damage of towers over base boundaries.
![11  towers](https://github.com/user-attachments/assets/e40d6a89-3fea-46a1-97bd-6e02599bf72f)
13. **Place Additional Structures**: Situate the factory, nuker, observer, and extensions. Placement for the observer is less critical, so anywhere works. At this stage, I changed lab position a bit and marked them as green square.
![12  nuker, factory, observer, extensions](https://github.com/user-attachments/assets/c6e8da9c-9b8f-47f4-9972-131c51ffd832)

## Examples and Visuals
Here are some examples created by my bot. The upgrade area is marked with a yellow square, labs with a green square, and towers with a red square.
![13  example(1)](https://github.com/user-attachments/assets/92005d2c-a093-45f3-ba20-cb7f6aad77c9)
![14  example(2)](https://github.com/user-attachments/assets/07b63b20-1057-4d9d-9d69-5198737e68cd)
![15  example(3)](https://github.com/user-attachments/assets/2cb0b637-7678-46e2-bba4-7d1cc9376b99)
![16  example(4)](https://github.com/user-attachments/assets/ef6e5ac0-cf8f-4c64-96b2-71e28c1d7f0f)
![17  example(5)](https://github.com/user-attachments/assets/aea38274-390a-4b1f-bbe8-555220f3d320)
Automating base planning streamlines your development process and lays a solid foundation for your Screeps bot to thrive.
