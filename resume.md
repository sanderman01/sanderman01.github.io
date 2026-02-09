---
title: Resume Alexander Verbeek
layout: resume
date: 2025-02-08
---

# Alexander "Sander" Verbeek, Game Developer

<img src="{{ site.url }}/assets/images/portrait.jpg" style="float: right"/>

Year of Birth: 1989  
Citizenship: The Netherlands  
Address: Fogdevreten 8, LGH 1202, SE-171 65 Solna, Sweden  
Portfolio: [http://sanderman01.github.io/portfolio](http://sanderman01.github.io/portfolio)  
Email: [sanderman@gmail.com](mailto:sanderman@gmail.com)  
Phone: +46 76 004 39 54  

Coder with a strong passion for video games. Gamedev since 2012.

## Notable Projects
* [Den of Wolves](https://www.denofwolves.com/en) Co-Op Heist FPS (10 Chambers)
* [GTFO](https://gtfothegame.com/) Hardcore Co-Op Horror FPS (10 Chambers)
* [Escapists 2](https://www.youtube.com/watch?v=qMj-bAyHqq4) Escape RPG port to iOS, Android (Codeglue, now part of Behaviour Interactive)
* [SpeedRunners](https://www.youtube.com/watch?v=BDGzN0fRQI0) Racing Platformer port to XBoxOne, Fuze (DoubleDutch Games)
* [iO]((https://store.steampowered.com/app/324070/iO/)) Physics Puzzle Platformer. My very first professionally released game.

## Strengths and Weaknesses
* Jack-of-many-trades code-wise. Adaptable.
* Persistent, focused problem solver
* Able to perform under pressure
* My approach is deliberate and thorough
* My ability to focus and deep dive into investigating an issue or topic helps me solve complex problems, though I need to take care to avoid tunnelvision.
* I sometimes struggle with ambiguously defined goals, tasks or expectations.

## Interests
I like to experiment to learn new techniques, technologies or skills, though finding time to do so can often be difficult when there are so many fun video games to play and books to read. Other hobbies include reading Fantasy and Science-Fiction novels, and pen & paper roleplaying games.

## Skills and Experience 
**Languages**
* C# (expert)
* C++ (proficient)
* Cg\HLSL shaders (proficient)
* Rust, Python, Java, Haskell, SQL, etc.  (prior experience)

**Game Engines**
* Unity3D (expert)
* Unreal Engine, Godot, Bevy (prior experience)

**Domain Experience**
* Gameplay Systems: interactions, stamina, gun behaviours and recoil, networking, etc.
* Game AI: Pathfinding and object avoidance, (see e.g. Flyers in GTFO) behaviour trees, flocking, etc.
* Destruction Systems: Applying physics forces on hit or collision, swapping out entities and other effects on trigger.
* UI design and implementation
* Editors and procedural generation: mesh topology, terrain, fractal noise, splines, scene optimization tools, etc.
* 3D Dot plots and 2D heatmap visualizations of playtest data for analysis e.g: Which areas in the scene do players frequent? Where do players or npcs get downed frequently?
* Graphics: vertex, fragment, geometry and compute shaders, post processing effects, ray marching, etc.
* Runtime profiling for performance and memory usage, and custom tools for capturing profiling data at the right moments and plotting/visualizing to gain insights.
* Backend services communication and integrating with systems such as
Steam, Xbox, PSN, PlayFab, Twitch, etc.
* Tech Writing: Documentation, code guidelines, asset production guidelines, proposals, diagrams.

## Employment History
#### Gameplay Programmer at 10 Chambers, 2021 to 2026

Designed and implemented various gameplay and other systems for Co-Op FPS games GTFO and Den of Wolves. I have worked on the teams: Core Tech, Player Journey, and Backend.
Some notable accomplishments:

##### Den of Wolves Destruction System:
Goal was to give tech artists tools to create destructible objects, and to allow various effects on trigger, such as object replacement with more detailed destroyed objects, and cheap primitive network syncing. To avoid load on dedicated servers, Destruction was to use custom sync rather than our standard NetEntity sync.
Achieved with a client-only destruction approach. Client sim sends messages to server and other clients indicating destroyed objects. Server acts as message broker and bookkeeper for an array where each bit represents one destructible object.
When state changes, or a new client joins, this array is sent and clients locally trigger each destructible object and its effects. Only trigger state bit is synced.

##### GTFO Flyer pathfinding:
Goal was to enable free movement throughout the entire volume of the rooms of the complex.
Achieved efficient pathfinding through volumes by dividing the level into multiple regular grids based on rooms, creating a sparse data structure. I used hierarchical pathfinding.Combined with clustering cells and bitpacking this allowed low memory usage, high data locality, and high performance. Each gridcell had a 4bit distance value, creating a distance field, to allow pathing for various size flyers.

#### Programmer at Codeglue, Rotterdam, 2017 to 2021**  
Various porting projects and in-house projects: Most challenging was Escapists2 to mobile as it required extensive networking changes and extreme optimization efforts. 
The Shadowrun port was another large project requiring extensive UI changes.
Various other in-house ip projects and gamejams.

#### Programmer at DoubleDutch Games, Hilversum, 2016**  
Porting SpeedRunners to XboxOne and Fuze.
Integrating platform systems such as: networking, matchmaking, application lifecycle management, cloud saves, achievements, voice chat, etc. 
Technical Requirements (TRC) implementation, testing and bugfixes. 

#### Programmer at Righteous Games, Eindhoven, 2015
Implementation and optimization of the Willem II Passage Occulus Rift VR sim and attached Arduino bike sensor system.
Designing and implementing gameplay for various logic and spatial awareness puzzles for an in-house ip puzzle-detective exploration game prototype targeted to tablets.

#### Indie. iO, 2012-2014
 Collaboration with Gamious, Amsterdam. Developing level editing tools, UI, and other game systems. Drafting and implementing levels. Conceptualizing novel ways to combine existing game mechanics. Developing control schemes for gamepad and touchscreen. For PC, Mobile and Ouya.

#### Internships: Little Chicken, Amsterdam 2012, Blewscreen, Tilburg 2010  
#### Education: HBO-ICT + Game Design and Technology at Fontys Eindhoven (Bachelor of Science) 