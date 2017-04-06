---
layout: post
title: The First Prototype
date: 2012-03-11 13:39
author: sanderman0
comments: true
tags: [game design, game development, internship, little chicken, persistence, research, research subject, software-development]
---
As planned, I have been working on my first prototype for my persistence layer. This prototype will be integrated into a current project with a deadline so it needs to be simple.

I have chosen the most promising methods for dealing with each sub-problem which I have identified in my earlier preliminary research.
<h1>Extraction</h1>
By extraction I mean the method by which the relevant data is collected and extracted from the current state of the game. I found three candidate solutions to solving this:
<ol>
	<li><strong>Domain model:</strong> Have all game objects contain references to separate data objects.</li>
	<li><strong><span style="text-decoration:underline;">Reflection</span>:</strong> Have all game objects use annotations to mark data they want serialized, and extract using reflection.</li>
	<li><strong>Manifest:</strong> Keep all data separate in a single object graph, commonly called a manifest</li>
</ol>
I decided that reflection is the most promising candidate for the first prototype, since it would enable a lot of flexibility and not require a major restructuring of the already existing code base. It results in a slightly more complicated persistence layer but is offset by the ease of development for the user. It was inspired by the way object relational mapping is commonly used in Java and to a lesser extend .NET systems.

The domain model and manifest solutions would result in a simpler persistence layer, but would require a lot of changes in the existing code base to supply the relevant data.
<h1>Serialization</h1>
I am working in Unity, which means I have access to most functionality in .NET implemented by Mono, including serialization.
<ol>
	<li><strong><span style="text-decoration:underline;">XmlWriter/XmlSerializer</span>:</strong> serialize to XML</li>
	<li><strong>BinaryFormatter:</strong> serialize to binary stream</li>
</ol>
The choice of serialization method to use is sometimes very dependent on the situation, but usually comes down to preference. In this case I was given to understand that the data would be sent to a server later, so encoding to text was recommended. Xml was the logical solution to use in this case.
<h1>Storage</h1>
During testing, storing data on the filesystem makes analyzing it and debugging easier, but later the data should be sent to a web server connected to a database. I decided to make the system flexible by creating interchangeable modules to change the behavior when needed. I made one module to save to the filesystem using a FileStream and another to interface with a different system handling the remote server.
<h1>Reconstruction</h1>
To prepare for reconstruction, I made sure to structure the serialized data in such a way as to be able to keep it apart later and still know where everything belongs. This is why I gave every entity a unique identifier. This allows the system to restore all the data to where it belongs. The actual restoring of the scene can by a combination of the following methods:
<ol>
	<li>Automatically save and restore entire scene hierarchy related to the entity</li>
	<li><span style="text-decoration:underline;">Save entity and use custom methods per entity for restoration after loading the data.</span></li>
	<li>For dynamic entities that should be instantiated, the factory pattern is a good approach.</li>
</ol>
I decided to go for the second option for two reasons. The most important reason is that option one would make the persistence layer very complex, and I do not have the time right now to implement such a solution, while the sacrifice in usability is very small. Each entity can easily handle its own initialization after being loaded with data by implementing OnAfterLoad and other methods. The other reason is that the first option might lead to a lot of redundant data, which might create storage issues and would introduce a lot of places where things could go horribly wrong.
<h1>To summarize</h1>
I have chosen the best solutions for the sub-problems identified and started implementing the first prototype. Initial results seem very promising. The prototype has already been integrated into the main project to have actual data to save and it is performing nicely. There are still a few kinks to work out, and a few glaring shortcomings, but it will serve for the current purposes.
