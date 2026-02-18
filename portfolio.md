---
layout: default
title: Portfolio
date: 2026-02-18
---

Ever since I started out in this industry I have worked on many projects. Some of which that I am particularly proud of receive a spot on this portfolio.

---

# Den of Wolves

<iframe width="455" height="256" src="https://www.youtube.com/embed/CCUnDA8BpIU" frameborder="0" allowfullscreen></iframe>

Den of Wolves is a co-op heist FPS. In this techno-thriller you and your friends operate as criminals for hire in the conflicts between rival corporations in Midway City. Gather your crew, design the plan, gear up and execute the heist.

This project is meant to be a spiritual successor to Ulf's previous projects, and is also inspired by movies such as Heist and Inception.

We started on pre-production for Den of Wolves in 2022.

##### Structure
Me and Stanislav Eremeev set up the Den of Wolves project during early pre-production. The two of us together with Ulf Andersson, decided to build on Unity's Data Oriented Tech stack. (DOTS) We knew this was a bit of a gamble, as these Unity packages were still experimental at the time.

Gameplay simulation ran on systems inside ECS worlds, with client world handling local gameplay simulation and presentation, and authoritative server world handling authoritative networking stuff. Using ECS worlds allowed us to easily run both server and client worlds within the same Unity Editor process to test client-server networking. This helped speed up testing during development as there was no need to spin up another Unity Editor process or create a build to test multiplayer functionality in a networking context.

The ECS paradigm allowed us to enforce very strict isolation between client and server worlds state within the same process, which would otherwise be difficult to achieve in a traditional Object Oriented Unity GameObject paradigm.

ECS also allowed us to take advantage of Unity's Jobs system and Burst compiler for performance in many game systems. 
A side benefit: Quick level load in release builds. Loading entities from flat tabular data (a few memcpy ops) is much faster than deserialising GameObjects and their component objects.

Unity's DOTS stack also created new difficulties. A lot of Unity systems and APIs were still traditional GameObject style and cumbersome/expensive to access from within ECS systems or jobs and vice versa from the main-thread. Baking assets into Entity Scene format also introduced extra complexity and during development we incurred a lot of time for scene baking before play mode. 

Overall I am of the opinion that DOTS ECS is good for the future of game development in Unity, but current transition era is problematic. Also this paradigm is better suited for some types of games than for others. It could be argued that the general gameplay logic of an FPS is not the best use-case for ECS.

Besides ECS, I proposed using the following solutions in the project:
* VContainer for dependency injection.  
(this paradigm was quite succesfull at making dependencies between systems more clearly declared and keeping code spaghetti to a minimum)
* ReWired for input mapping and controller support  
(I had prior experience with this product and had more faith in it than the new Unity input package at that time)
* UniTask for lightweight async tasks, for communication to our backend and other remote services.  (Helped writing async code and avoid/reduce callback hell in code handling remote requests and responses)

##### Destruction System
I designed and implemented a destruction system with the goal to give tech artists flexible tools to create destructible objects prefabs, and to allow various effects on trigger, such as object replacement with more detailed destroyed object models, and cheap primitive network syncing. To avoid load on dedicated servers, Destruction was to use custom sync rather than our standard NetEntity sync.  (diffing that many entities would have been too expensive, at least early in development of our networking system)

Achieved with a client-only destruction sim approach. Client sim sends messages to server and other clients indicating destroyed objects. Server acts as message broker and bookkeeper. When a new client joins, the data keeping track of which objects are destroyed is small enough be easily sent to other clients. (basically a bitarray)

This partial syncing was fine as long as destruction did not significantly affect bullet hits, player collisions or health. Ironically it was later decided to pivot to Peer-to-Peer networking. In hindsight, I could have done a destruction system with higher fidelity network syncing and greater flexibility.

##### Misc
Besides the above, I also worked on many other areas such as:
* Gameplay Interactions. (e.g. doors)
* Weapon Recoil. (a deceptively complex problem)
* Volume Pathfinding. (based on the flyer pathfinding I had created for GTFO previously)
* Gameplay Graph. (in-house visual logic tool with ECS support, similar to Unreal's Blueprint)
* Backend communication. Initially to our own backend using GRPC and later PlayFab.
* In-game Profiler Recording tools, and out-of-game tools for visualizing profiling data.
* Data Visualization. (e.g. In which level areas do player get downed often)
* Logging from dedicated servers running in Kubernetes in Google Cloud Platform.
* Testing Framework
* Debugging crashdumps

# GTFO

<iframe width="455" height="256" src="https://www.youtube.com/embed/dhMw5Zb6hZs" frameborder="0" allowfullscreen></iframe>

GTFO is a hardcore cooperative horror shooter that throws you from gripping suspense to explosive action in a heartbeat. Stealth, strategy, and teamwork are necessary to survive in your deadly, underground prison. Work together or die together.

I was recruited to 10 Chambers in April 2021 to help develop GTFO to its 1.0 release in December that year, and beyond. I worked on the following areas:
* Various gameplay bugfixes and diagnosing crash dumps
* Improvements and fixes for the Glue Gun
* Designed and implementated a player stamina system  (Goal was to disincentivise kiting enemies during combat, but also not to hinder non-combat exploration of the complex outside of combat)
* Collaboration with Gameplay Designers to implement functionality for the abilities of the end boss of Rundown 6.0, an enemy commonly known as the Kraken or Nemesis.
* Designed and implementated a pathfinding system through 3D volumes to be used by flying enemies of varying sizes.  
I started out using a oct-tree approach, but then replaced that with a simpler system where each room has its own corresponding regular grid and each cell has a distance value to the nearest wall or other solid obstacle.  Using multiple regular grids covering rooms rather than one big grid covering the entire level resulted in something resembling a sparse data structure, while retaining simplicity, performance, and limiting memory usage. For pathfinding I used a hierarchical approach, calculating high-level paths from room to room, and lower level paths only within the room and to the players or to the door/passage into the next room, both using a variant of A-Star.

We released GTFO 1.0 on Steam in December 2021.

# Shadowrun Trilogy

<iframe width="455" height="256" src="https://www.youtube.com/embed/IinkPsj5taA" frameborder="0" allowfullscreen></iframe>

Shadowrun is a tactical role-playing game which takes place in the science fantasy setting of the Shadowrun tabletop role-playing game.

The Shadowrun Trilogy project was to port the three Shadowrun games to the consoles XboxOne and Series X, PS4 and PS5, and Nintendo Switch, as a compilation game. This involved:

* Upgrading the project to the latest version of the Unity game engine and getting it to run on console systems.
* Replacing the old UI based on the deprecated and no longer maintained EGUI system with the newer uGUI.
* Overhauling the design and functionality of all UI screens making them suitable for control using gamepads or other controllers.
* Integration with console functionality and systems such as user profiles, cloud saves, IME, suspend/resume, etc.
* Various optimizations to improve game performance on systems with limited memory and gpu performance.
* Technical requirements work (eg. XRs) for compatibility and title certification on consoles.

# Escapists 2

<iframe width="455" height="256" src="https://www.youtube.com/embed/oDPtBf6P1wQ" frameborder="0" allowfullscreen></iframe>

I joined Codeglue in September 2017. Over the course of several months, I was assigned to a number of porting projects, such as Castaway Paradise, Corpse Party: Blood Drive, and Descenders.
My first large project at Codeglue became the task of porting Escapists 2 from PC to iOS and Android for Team17, in a team of several people.

Escapists is a top-down game where the player takes the role of a prison inmate with the goal to break out of prison. Each prison level has a routine with daily activities for prisoners and players have a lot of options to cause chaos in the prison complex and to design and execute elaborate escape plans. The game also includes networked multiplayer, allowing multiple people to play together and co-operate or compete with each other.

This project's story is one of hardships and perseverance.

The Escapists 2 project turned out to be extremely challenging and severely underestimated/overscoped for the time allocated due to a variety of factors, but mainly due to the following constraints:
* Difficulties communicating with the client. (this meant important issues/decisions requiring client input could not be adequately addressed in a timely manner and would crop up later at inopportune moments)
* Targeting a specific and very old model of iPhone. (This meant _extremely aggressive_ memory and performance optimizations, many of which required extensive changes to game systems, which would not have been neccesary, if an even slightly more recent device had been targeted)
* A requirement that the game should work on local wifi hotspots not connected to the internet. Eg. kids playing in a car back-seat on vacation. (The game was built on Photon Pun which is designed to always connect to a Photon Cloud. We had to get _very_ creative with the networking code to hack in offline LAN multiplayer capability)
* Implementing not just one, but several different control schemes for touch devices. (this added to the already huge workload)

The memory and networking requirements especially meant we were forced to dig deep and make very extensive -and risky!- changes to core systems within the game, and in the Photon Pun networking code. Many of these changes caused bugs, which of course needed to be fixed. The game did not include any test suite. In hindsight, we should have perhaps added regression tests ourselves. Combined with the communication difficulties with the client, these problems turned out to be a recipe for a lengthy development hell.

Nevertheless, we were eventually able to implement all the changes needed, squash most of the bugs caused by the overhaul, and satisfy the client. The game has since been released on iOS and Android as [Escapists 2 Pocket Breakout](https://apps.apple.com/us/app/escapists-2-pocket-breakout/id1356167732).

---

# SpeedRunners

<iframe width="455" height="256" src="https://www.youtube.com/embed/BDGzN0fRQI0" frameborder="0" allowfullscreen></iframe>

Speedrunners is a multiplayer side-scrolling game where you try to run faster than the other guys until they drop off-screen and they explode. It is great fun. It is huge! Trust me. You'll love it!

My contract at DoubleDutch Games was to help them port their game SpeedRunners to the consoles. There are a lot of issues to tackle when porting to these platforms, even when using a reasonably portable game engine like Unity3D. The game has a rather large legacy codebase, being originally created in XNA and later ported to MonoGame and then to Unity.

Personally, I worked on the Fuze and Xbox One ports, adapting existing code or writing new code to integrate with various systems on these platforms. Platform holders are strict about their Technical Requirements Checklists. They require a lot of changes to the game to ensure it interacts with existing systems on the platform and to ensure the game behaves as expected. This involved such things as:

* Application Lifecycle Management  
  *eg. constrain/suspend/resume*
* Active User Management and User Privileges handling  
  *eg. ensure that game state, local game saves, cloud saves and other user data is handled correctly, at all times. This includes many unpredictable situations such as system users being switched out mid-game or a user being forcefully logged out due to remote events, among others.*
* Peer to Peer networking
* Matchmaking
* Voice Chat
* Friends Lists and Presence Management
* Events and Achievements systems within the game, and configuration in the back-end
* Ensuring correct platform and language dependent terms and imagery within the game  
  *eg. for popup notifications or to refer to things like controller buttons*
* Reading messy documentation about above mentioned platform APIs and conventions.
* A huge amount of time bughunting and fixing
  
Sometimes this involved writing translation wrappers around platform specific systems, so that the base game could access them in using a platform independent interface. In many other situations, we were not so lucky and invasive changes in the core of the game were required. From new UI screens to large changes of existing systems.

Porting an existing PC game to consoles can be a huge amount of work. Working on this project has definitely give me more perspective on games ported to/from console not only as developer but also as a gamer.

[SpeedRunners is developed by DoubleDutch Games and published by tinyBuild.](http://www.tinybuild.com/speedrunners)
  
---

# iO
<iframe width="455" height="256" src="https://www.youtube.com/embed/nfQKWwc-op8" frameborder="0" allowfullscreen></iframe>
<iframe width="455" height="256" src="https://www.youtube.com/embed/5bc2q0EAOXA" frameborder="0" allowfullscreen></iframe>
![iO screenshot](assets/images/io/Screenshot iO_2.jpg)
![iO screenshot](assets/images/io/Screenshot iO_3.jpg)

[iO](http://gamious.com/io) started as a game prototype originally conceived and created by an ad hoc team of me and four others, during the Global Gam Jam of 2012 in Utrecht. Back then it was simply called Size Matters. The publishing company Gamious approached us and told us they wanted to work together with our team to take this game to the next level. What followed was a lengthy development process to expand upon the game with new game mechanics, extra content and to add the features and polishing neccesary for publishing to the major digital game stores.

This took way longer than any of us expected. Most team members had regular daytime jobs, and I was basically the only member working on the project full-time. We were also physically distributed throughout the country, so for communication, we were forced to rely on tools like trello and on schedules meetings over Skype. Keeping the project on track and team members motivated was extremely difficult. The project took too long really, which lead to all kinds of troubles in the development process. Despite these issues, we persevered and launched the game to the mobile stores and Steam and Ouya. I then took some distance from the project in order to recover some sanity points. Gamious eventually also ported and released the game on XBox One and Playstation 4.

iO is a unique mix between platforming, physics puzzles and racing. The simple colorscheme and smooth curves are remniscent of Tron and Sonic the Hedgehog. The music has a zen feeling to it, which is welcoming in the early easy levels, and which will certainly help you to keep your calm in some of the later levels, which can be extremely challenging.

I am rather proud of the game mechanics, which are very simple, yet allow for a ton of depth in the challenges that can be created, as evidenced by the huge number of levels present in the game. The player can provide torque to roll left or right, and he can grow or shrink. Changing size in this manner also influences properties like mass and friction. This allows for many interesting tricks. [Take a look at how I approached iO level design.]({% post_url 2015-11-10-on-the-level-design-in-io %})

I am also particularly satisfied with [the level editing tools I created]({% post_url 2013-02-06-creating-a-spline-level-editor-in-unity %}) within the Unity3D editor by writing custom editor extensions. This allowed even non-technical members of the team to have a hand in creating levels. These tools made it very easy to create and manipulate level geometry using bezier curves, with appropriate triangle meshes and colliders being generated on-the-fly procedurally. Since all of this happened inside the Unity3D scene editor, testing levels was simply a matter of hitting play mode and allowed fast iteration.

[iO is published by Gamious and now available on Steam, Windows Store, Xbox One, Playstation 4, Google Play, and App Store.](https://store.steampowered.com/app/324070/iO/)

![iO screenshot](assets/images/io/Screenshot iO_4.jpg)
![iO screenshot](assets/images/io/Screenshot iO_0.jpg)
![iO screenshot](assets/images/io/Screenshot iO_1.jpg)

---

# Other Projects

These projects tend to be small but interesting in some way. **Source code is available.** If you want, you can clone the repo, open it in Unity, and hit play to see the code in action.

---

## NBody Simulation on GPU

An n-body system is a system where many bodies interact with each other. Typically, each particle interacts with every single other particle in the simulation. Simulating and rendering such a system makes for an interesting problem. This project implements an n-body system with gravity on the GPU using compute shaders.

[This project is available on github.](https://github.com/sanderman01/unity-nbody)

---

## Super Wavy Tag Team Crowdsurfing

<iframe width="455" height="256" src="https://www.youtube.com/embed/kdmH_0UMJiM" frameborder="0" allowfullscreen></iframe>

*You finally get to see your favorite band live, you are so close to the stage that you can almost touch them, and then, the singer jumps to surf the waves formed by the hands of his biggest fans, yes that's you! It is your duty... No! it is your DESTINY to carry him as far as possible without dropping him to the ground, for the glory of ROCK!*

A game created recently at the Global Game Jam 2017 in Amsterdam. The theme was Wave, and we settled on the idea of waves in a crowd, such as the crowds at a sports stadium or music concert. In this co-operative 4-player party game you and your friends play the crowd. A rockstar jumps off the stage and it is your job to help him crowdsurf safely and without 'accidents.' It is a very simple game but immensely fun, with lots of potential for hillarity. When playtesters are laughing so hard that it hurts, then you know you've got a fun game.

[Global Game Jam project page](http://globalgamejam.org/2017/games/super-wavy-tag-team-crowd-surfing)  
[This project is available on github.](https://github.com/sanderman01/ggj2017-crowdsurf)

![GGJ event photo](assets/images/crowdsurf/crowdsurf-jam-award.jpg)

---

## Planet Search

<iframe src="https://player.vimeo.com/video/104621204" width="455" height="256" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>

A game created during the 2014 Pillo Jam. Goal of this game jam was to create game controlled using devices with sensors embedded in real live pillows. This is a 2-player game. Each player controls a single rocket thruster, which will tend to turn the spacecraft to the left or right. Careful coordination is required to fly towards your intended destination. Or you can relax and simply fly around to see where you happen to end up.

[This project is available on Bitbucket](https://bitbucket.org/sanderman/pillojam)

---

## Pathfinding demo

A basic A* pathfinding demonstration. This includes a pseudorandomly generated cloud of nodes, representing a galaxy. Nodes are linked by pathways, similar to the hyperlanes in 4x strategy games such as Stellaris. The user can click on two nodes and a path from one to the other will be calculated and highlighted.

[This project is available on github.](https://github.com/sanderman01/pathfinding-demo)

---

## Public Space Invaders

<iframe width="455" height="256" src="https://www.youtube.com/embed/XTN57S6kZ7g" frameborder="0" allowfullscreen></iframe>

A game created during the 2012 Games4Health Jam in Eindhoven. Jam goal was to create a game adding extra life to public spaces and to stimulate excercise and movement. We used a top down-camera and computer-vision (OpenCV) to map the locations of players in a large physical area to the locations of player avatars within the game. This allowed for anyone to join in playing the game simply by walking into the play area.