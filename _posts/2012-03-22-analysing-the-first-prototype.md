---
layout: post
title: Analysing the first prototype
date: 2012-03-22 14:46
author: sanderman0
comments: true
tags: [game design, game development, internship, little chicken, persistence, research, research subject, software-development]
---
The past weeks I have been working on working on some general gameplay functionality for the main project, in addition to further integration of the first prototype of my persistence layer. It is now possible to store the game state in a save slot on a server. I have also analyzed the current strengths and weaknesses of the system. The following diagram shows how it is designed:

<a href="http://sanderman0.files.wordpress.com/2012/03/persistence-layer-prototype-1-design.png"><img class="aligncenter size-full wp-image-120" title="Persistence Layer Prototype 1 Design" src="http://sanderman0.files.wordpress.com/2012/03/persistence-layer-prototype-1-design.png" alt="" width="584" height="461" /></a>The most important parts are the IEntity interface which is implemented by game entities and enables them to mark fields to be persisted and to receive signals after loading and other persistence events. The bulk of the work is handled in the implementations of IStreamEncoder, of which currently only XmlEncoder is implemented and uses reflection and the XmlWriter and XmlSerializer classes to serialize the entities and their fields to an xml document. Which can be saved to a file, or somewhere else, by the IPersister.

I described the relevant problems and my chosen solutions in my previous post so I will not go into those here and now. I was mostly on the right track, and the overall system is working fine, though there are still quite a few points of improvement.
<h1>Strengths</h1>
<ul>
	<li><strong>Seperation of responsibility:</strong> Serialization and storage details are abstracted away from the rest of the game logic.<strong></strong></li>
	<li><strong>Simple and Intuitive:</strong> Adding extra persistent game entities is easily done by implementing IEntity and marking the relevant fields and properties with the PersistField attribute.<strong></strong></li>
	<li><strong>Convenient:</strong> It is possible to persist fields defined in an object related or attach to the entity by wrapping them in a persistent property.<strong></strong></li>
	<li><strong>Flexible:</strong> different implementations of IPersister can easily be switched out to change the overall behaviour of the system.</li>
</ul>
<h1>Weaknesses</h1>
<ul>
	<li><strong>Dynamic entities:</strong> The current system collects all entities in the scene during saving, and extracts their data. Upon loading, all entities in the scene are again collected, and filled with the data that was saved earlier. This works for scenarios where the state of entities changes, but it does not work in scenarios where entities have to be created, or removed to reach the saved state. We used a workaround in the form of a manager entity to handle the state of a variable number of other game elements. Recreating entities at the right place in the scene and hierarchy would be desirable. This also will require a way to instantiate game entities without needing references to prefabs.</li>
	<li><strong>Dependencies:</strong> Some entities turned out to have dependencies with others and expected those to have been initialized already. I quickly solved this by adding a priority to IEntity to govern the order in which OnAfterLoad is called on them. This is perhaps not the best solution. Priority based on location in the scene hierarchy might be an intuitive way to handle this, but is less flexible. Food for thought.</li>
	<li><strong>Integration:</strong> Integrating with existing systems turned out to be a bit trickier than expected. The save methods do not take parameters so options like save names needed to be set in the current IPersister while ideally, only the IPersistenceManager should be accessed once the system is configured. This part of the design could be reworked.</li>
</ul>
<h1>Other Comments</h1>
<ul>
	<li><strong>IEntity:</strong> This interface now contains four methods implemented by entities to receive signals when needed on persistence events. These should be optional but the interface requires an implementation, which can be irritating. This could be changed.</li>
	<li><strong>Design:</strong> As of this moment the bulk of the work happens in the XmlEncoder, which will require duplicating functionality when different encoders are desired. The extraction of data and serialization of data should be separated.</li>
	<li><strong>IEntity:</strong> My colleague offered another suggestion to handle the extraction problem. Instead of code annotations, this would involve an object with a list of names of fields to be persisted, which could be attached to a game entity. In Unity this could be a component, granting the user the ability to set values to save in the inspector. It is an interesting idea which merits further investigation.</li>
</ul>
<h1>Conclusion</h1>
The current prototype is quite successful. It does the job and does it reliably, and has kept the persistence implementation of most game entities relatively simple. It has proven that the chosen solutions work for this problem. There are still aspects where it falls short, which should be addressed in the following iterations of new prototypes. Handling dynamic game entities and dependencies should have the highest priority right now.
