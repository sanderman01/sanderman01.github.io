---
layout: post
title: Evaluation
date: 2012-05-30 15:32
author: sanderman0
comments: true
tags: [game design, game development, internship, little chicken, persistence, game saves, research, research subject]
---
<strong>[update]</strong>
I've since completed this project and finished my internship at Little Chicken and graduated from school with a <em>Bachelor of ICT and Game Design and Technology</em> degree.
<strong>[/update]</strong>

Since the last update I have mainly been working on side projects. Recently my colleague finally had time to do an evaluation of my persistence layer project. The results are very satisfying. There were only a few comments on issues meriting further consideration.
<h1>Automatic identifiers</h1>
Currently, every instance of an entity needs to be assigned a unique identifier to be persisted correctly. This is cumbersome to manage for the game developer and it might lead to mistakes and confusion. It was suggested to change this so that entities might be assigned a unique identifier automatically by the persistence layer.

This was something I did not consider at first, because I wanted the flexibility for the developer to be able to choose his own identifiers, but the idea is sound. My colleague pointed me to the GetInstanceID method on components which is ideal for this sort of thing and which I had previously overlooked somehow. There is only a trivial change required to implement this change.
<h1>Hierarchy</h1>
With GetInstanceID it should also be possible to distinguish different Transform components, making persisting and restoring parts of the hierarchy an option again.

Unfortunately all the other issues with the hierarchy that I noticed before this have not disappeared. Restoring the hierarchy will still be a very complex issue, with a lot of potential for bugs and edge-cases that will be difficult to deal with, making the overall system less reliable.

I still feel that this feature is more of a 'nice to have', than something essential, so for now I will focus on polishing the rough edges, writing documentation, and the finishing touches on my thesis for school.
<h1>Multiple Entity Components on a spawned GameObject</h1>
This was a question which confused me a bit. Why would you ever put more than one entity component on the same prefab? This way they can not be separated so they are effectively the same entity, which defeats the point of having multiple entity components. I could only conclude that it was a desire to organize things.

Due to the way this system is designed, dealing with this strange edge case is not possible, as the system would try to spawn two game objects from prefab while there should actually be only one.
<h1>Conclusion</h1>
While there are still a few things to be desired, the overall system works fine and fulfills its purpose. All that remains is to polish some rough edges and write documentation. I will also put the finishing touches on my thesis for school, which is nearing the deadline. After that, I might still have time to look at handling the hierarchy.
