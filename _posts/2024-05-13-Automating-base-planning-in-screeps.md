---
layout: single
title: "Automating Base Planning in Screeps – A Step-by-Step Guide"

toc: true
toc_sticky: true

comments: true
---

## Why Base Planning?

Planning the layout of your base in Screeps might seem unusual at first, but it’s a crucial step in automating your bot. It’s also one of the most challenging tasks to tackle on your own from scratch. While you don't necessarily need to fully understand the algorithms involved, you should at least know how to utilize them effectively. So, here’s a step-by-step guide on how I automate my base planning. Let’s dive into it.

## The Process

1. **Utilize Distance Transform**  
   Start by employing the distance transform algorithm to identify open spaces in the room. Typically, my bot requires a 5x5 square to begin, so I target positions with a distance of 3 or larger. Some bots, especially those employing bunker-style base planners, may require squares larger than 10x10. Plan ahead for the desired size of your base. The distance transform code is available [here](https://github.com/sy-harabi/screeps-algorithgm-utils/blob/33a0a406d86ed0a916d540340b3d07e3f5992065/utils.js#L10).  

   ![Screenshot: Distance transform result](https://github.com/user-attachments/assets/fb8c5f03-9e0a-4579-82ff-a21a124a5380)

1. **Select Starting Position**  
   Choose an open area you think is best for your base. I prefer locations close to both the controller and energy sources, especially proximity to the controller. But remember, everything is up to you!  

   ![Screenshot: Selected starting position](https://github.com/user-attachments/assets/88ee32ea-99ea-445b-ba3f-bbf76c26d37c)

1. **Place Core Structures**  
   Begin by placing core structures such as spawn, storage, and terminal. I surround the starting position with core structures using a 5x5 stamp for efficiency. Some bots utilize a stamp called "fast filler" to quickly fill extensions. The choice of design is up to you!  

   ![Screenshot: Core structure layout](https://github.com/user-attachments/assets/423a1593-08a4-4552-8b50-ee5cc6055e82)

1. **Designate Controller Upgrade Area**  
   Allocate a 3x3 area near the core structures for upgrading the controller. I made a mistake in the screenshot below — I didn’t properly ensure that every position was within range 3 of the controller. Don’t make the same mistake!  

   ![Screenshot: Controller upgrade area](https://github.com/user-attachments/assets/7b300577-b6a7-4a92-bfe4-28af168811cb)

1. **Implement Floodfill Algorithm**  
   Utilize the floodfill algorithm to categorize tiles by their accessibility. Lower numbers indicate proximity to core structures. The floodfill code is available [here](https://github.com/sy-harabi/screeps-algorithgm-utils/blob/33a0a406d86ed0a916d540340b3d07e3f5992065/utils.js#L115).  

   ![Screenshot: Floodfill result](https://github.com/user-attachments/assets/93ff0067-7c19-40c4-9835-65a4d8caf371)

1. **Allocate Tile Usage**  
   Assign tiles for essential structures such as extensions, labs, towers, factory, nuker, and observer. I personally prefer a grid layout similar to the "commie bot."  

   ![Screenshot: Grid layout](https://github.com/user-attachments/assets/11142e54-853e-458d-9040-a906dcb6e009)

1. **Position Labs**  
   Identify 10 suitable tiles meeting specific criteria, ensuring that there are two source labs within range 2 of every other lab. If it seems too complicated, it’s fine to use a predefined lab stamp.  

   ![Screenshot: Lab placement](https://github.com/user-attachments/assets/553cfa99-8983-415a-a137-63b9ec336b82)

1. **Establish Infrastructure**  
   Lay down roads from storage to energy sources and minerals. Place a container and a link at each source, and a container and an extractor at the mineral.  

   ![Screenshot: Roads to resources](https://github.com/user-attachments/assets/eba74c08-c41d-4c7d-8f05-52633f11b3ba)

1. **Optimize Ramparts**  
   Utilize the minimum cut algorithm to determine optimal rampart positions, then connect them with roads. At this stage, I noticed my earlier mistake and moved the controller upgrade area closer to the controller. The minimum cut code is available [here](https://github.com/sy-harabi/screeps-algorithgm-utils/blob/33a0a406d86ed0a916d540340b3d07e3f5992065/utils.js#L204).  

   ![Screenshot: Roads to ramparts](https://github.com/user-attachments/assets/1114dd73-d54a-4f31-9ebc-da26b38f85ee)

1. **Tower Placement**  
   Position towers to maximize their coverage. Personally, I prioritize maximizing the minimum damage of towers over strictly defining base boundaries.  

   ![Screenshot: Tower placement](https://github.com/user-attachments/assets/e40d6a89-3fea-46a1-97bd-6e02599bf72f)

1. **Place Additional Structures**  
   Situate the factory, nuker, observer, and extensions. Placement for the observer is less critical, so anywhere works. At this stage, I also adjusted the lab positions slightly and marked them as green squares.  

   ![Screenshot: Nuker, factory, observer, extensions](https://github.com/user-attachments/assets/c6e8da9c-9b8f-47f4-9972-131c51ffd832)

## Examples and Visuals

Here are some examples created by my bot. The upgrade area is marked with a yellow square, labs with green squares, and towers with red squares.  

![Screenshot: Example base layout 1](https://github.com/user-attachments/assets/92005d2c-a093-45f3-ba20-cb7f6aad77c9)

![Screenshot: Example base layout 2](https://github.com/user-attachments/assets/07b63b20-1057-4d9d-9d69-5198737e68cd)

![Screenshot: Example base layout 3](https://github.com/user-attachments/assets/2cb0b637-7678-46e2-bba4-7d1cc9376b99)

![Screenshot: Example base layout 4](https://github.com/user-attachments/assets/ef6e5ac0-cf8f-4c64-96b2-71e28c1d7f0f)

![Screenshot: Example base layout 5](https://github.com/user-attachments/assets/aea38274-390a-4b1f-bbe8-555220f3d320)

---

Automating base planning streamlines your development process and lays a solid foundation for your Screeps bot to thrive.
