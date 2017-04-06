---
layout: post
title: Research on Persistence
date: 2012-02-26 11:37
author: sanderman0
comments: true
tags: [game design, game development, internship, little chicken, persistence, research, research subject, software development]
---
The past week I have been doing preliminary research on persistence in the context of game development in preparation to developing a standard system to handle this.

I have scoured the web and found quite a few sources on this topic, though most of them have not been very useful. Most questions and responses in the Unity community focus on the smaller problems like serialization. There is very little deeper discussion on the design of systems that should handle this. The same goes for most game development forums. It is commonly said that every game has different needs and thus persistence should be custom-built for every game. I am not satisfied with that, because I feel that most of them deal with fundamentally the same problems, just in different situations. In my opinion, a lot of this could be generalized.

The problem of persistence has been largely solved in enterprise software. There are advanced databases and other data stores available, with abstraction layers to ease development. Application functionality is divided into domain models for persistence and other parts for the rest of the functionality. This makes persistence a trivial problem for most software engineers.

This is usually not the case in game development. In my experience a lot of games are developed more in an ad-hock fashion than most  applications. For reasons of performance and ease of use, the data we want to save is often embedded in the scene in ways that make it difficult to get it out if persistence was an afterthought. Any Unity developer who has ever tried to save and restore entire game object hierarchies at run-time will know the difficulties I am talking about. Whether these designs are wise is another matter I will not go into here. In addition to this, there is the fact that different games use different storage methods. Some might use a save file on a local hard disk while others will save state to a web service or database. All these circumstances make persistence for games a touchy subject.

I took another close look at all the resources I had gathered and noted that the entire problem of persistence in game design can be summarized in a few sub-problems:
<ol>
	<li><strong>Extraction:</strong> Separating the data you want to persist from the rest of the game</li>
	<li><strong>Serialization:</strong> Encoding the data into a sequential format suitable for storage and back</li>
	<li><strong>Storage:</strong> Actually storing and retrieving the serialized data</li>
	<li><strong>Reconstruction:</strong> Reconstructing the scene or game state from loaded data</li>
</ol>
In a lot of situations, these problems might overlap or even blur as to be almost indistinguishable from each other. At all times every one of these problems is present in some way. Most resources I have found focused on serialization and storage. The more difficult problems are actually extraction and reconstruction, but often overlooked, which often causes problems in serialization.

For each of these problems I have found a number of ways to deal with the problem. I will choose the most promising solutions I have found and integrate these into my first prototype.
