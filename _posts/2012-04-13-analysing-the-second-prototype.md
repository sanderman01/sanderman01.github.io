---
layout: post
title: Analysing the second prototype
date: 2012-04-13 16:13
author: sanderman0
comments: true
tags: [game design, game development, internship, little chicken, persistence, research, research subject, software-development]
---
It's been a while since the last update. The last month I have been working more on the main project, which successfully passed an important deadline. The past weeks I have been working on my own research and assignment. The first prototype has been well received and with this second prototype, the project is slowly nearing completion.

The past few weeks I have been expanding on this to create the second prototype, or iteration, for my Unity persistence layer. I detailed the major weaknesses of the first prototype in my previous post, along with some other considerations. The most important problems were solved in the second prototype, but there were some difficulties as well and some problems turned out to be impossible to solve reliably.
<h1>Chosen solutions</h1>
<h2>Dynamic Entities Instantiation</h2>
The biggest weakness of the original prototype was the fact that it was only able to fill existing entities with persistent data from storage and not able to instantiate new entities in the scene if this was required for the game. Initially, a work around was used to mitigate this problem.

In the new prototype, this problem has been solved by the addition of the Type property and the IEntityFactory interface. Developers using this persistence layer can implement an entity factory and plug it into the system to add enable instantiating dynamic entities.

This solution was chosen because, depending on the game and the design and implementation of game entities, instantiating an entity is likely to be quite complex. In Unity, game entities are likely to be inherited from the MonoBehaviour class, which means that they can not be instantiated normally. Instead, they usually need to be instantiated from a prefab, together with meshes, colliders and other components they are composed of.

The type property allows the EntityFactory to select the right prefab, even though the entity classes of different prefabs might be the same. For example, a red car and a blue car might each have their own prefab, but both are a Car entity. Note that entities do not HAVE to be inherited from MonoBehaviour and can also be plain old c# objects. In that case the factory can just 'new' an object like normal.
<h2>Handling References between Entities</h2>
With dynamic entity instantiation, there occurs a related problem, namely the handling of references between entities. Since entities are instantiated elsewhere, we would need to override the serialization behavior to return references to these already existing entities. This is a very useful feature to have when dealing with collections of entities.

An appropriate example would be a Track entity which maintains a list of all TrackPiece entities.

This presented a bit of a problem because these references to entities might be embedded quite deep in the persistent data fields of an entity. This means that the serialization system needs to deal with this somehow. Whatever serialization mechanism is used needs a way to detect the presence of a reference to a game entity, and instead of trying to serialize it, must write the id of this entity, because the entity itself will be handled manually by the rest of the persistence layer. Upon loading, the serialization mechanism will again need to recognize that it should not attempt to deserialize references to entities in the normal way, but instead call back up to the persistence layer to obtain a reference to the entity with that id.

This was accomplished two of the three implemented encoders. In the JsonEncoder the JsonConverter class could be overridden to change the behaviour of the JsonSerializer.

Likewise, in the BinaryEncoder, the ISerializationSurrogate interface was used in a similiar way.

Unfortunately the XmlSerializer used by XmlEncoder did not support any feature like this and the more modern XmlObjectSerializer is not included in the version of Mono that ships with Unity, so this encoder does not support serialization of references between entities.
<h2>Restoring the hierarchy</h2>
With dynamic entity instantiation, another problem that crops up is the scene hierarchy. In an variety of situations, it might be needed to restore an entity to its proper location in the hierarchy.

This feature was attempted, but abandoned due to several reasons.

Writing the hierarchy data did not prove to be a significant problem. Recursion was used to filter the transforms of all entities and their parents. These were written to the save file and loaded again correctly.

Unfortunately restoring the hierarchy in the scene properly proved impossible to do because of limitations in Unity. This stems from the fact that individual transforms in the scene do not have any unique identifiers. Different transforms can have the same names which leads to unpredictable behavior when loading since different transforms can not be distinguished from each other.

The only way to implement this would be to assign a unique identifier to each transform upon saving and upon loading to load every single one back into the scene. Obviously this does not work in cases where some entities are already defined in the scene editor as they would be duplicated instead of merely filled with the right data. This solution would make saving and loading an all-or-nothing approach and does not fit with Unity's design philosophy in my opinion.

Another, less important reason to scrap this feature was that saving the entire hierarchy proved to save quite a bit of redundant data that might not be needed. Instead, having each entity manage it's own location in the hierarchy if relevant was considered a better option. This option is less friendly to the developer but offers a lot more flexibility to the developer, who can make his own judgment on how to handle this issue on a case by case basis.
<h2>Integration</h2>
To enable easier integration with other systems, a change in the design was necessary to make the method of storage more accessible, so that parameters can more easily be changed and data can be exchanged. The new design is further described in the following section.
<h1>Design</h1>
The design of the system was changed quite a bit compared to the initial design.

The IPersister and IPersister manager and implementing classes have been merged. This makes it easier to access the storage options or data from outside.

In addition, there is now an abstract base class StreamEncoder, which handles extracting the data from entities and instantiating them. Subclasses inheriting from StreamEncoder override abstract methods to implement serialization details and writing and reading all data to and from the stream. This separates the actual extraction and restoration work from the serialization method chosen, even though they are still tightly related.

The additional feature of handling references between objects required a change in the general algorithm for loading entities and restoring the scene. This now happens in two passes, first instantiating all entities, and then filling all entities with data and reconnecting references between entities.
<h1><a href="https://sanderman0.files.wordpress.com/2012/04/persistence-layer-prototype-2-design1.png"><img class="aligncenter size-full wp-image-142" title="Persistence Layer Prototype 2 Design" src="https://sanderman0.files.wordpress.com/2012/04/persistence-layer-prototype-2-design1.png" alt="" width="584" height="821" /></a>Evaluation</h1>
Since the original Workshop project is pretty much complete right now, testing these changes is better done somewhere else. Having a smaller project also helps with testing only the functionality of this system.

To test and evaluate this prototype, a new project was created containing a test scene with a collection of entities implemented to cover all functionality during testing. The following features were tested in the following ways:
<ul>
	<li>Persist static entities. Saving and restoring entities defined in the scene in the editor. Three moving Ball entities were placed into the scene and given persistent properties for position, orientation, scale and velocity.</li>
	<li>Persist dynamic entities. Saving and restoring entities instantiated at run-time. A BallSpawner entity was added to the scene which instantiates smaller ball entities on the press of a button.</li>
	<li>References. Saving and restoring persistent references between entities. The BallSpawner was given a list with references to all balls it spawned. Balls were given a reference to the last Ball they collided with. All of these references were rendered by drawing lines in the scene for easy visual confirmation.</li>
</ul>
Together this test scene and it's contents covered all features I designed into this system. Static Balls are correctly restored to the right position on load. Spawned balls are instantiated correctly and restored to the the right position as well. All references among entities are restored correctly.
<h2>Strengths</h2>
All strengths of the first prototype, plus the following:
<ul>
	<li>Support for dynamic entities</li>
	<li>Support for persistent references between entities</li>
	<li>A more flexible design with less code duplication</li>
</ul>
<h2>Weaknesses</h2>
<ul>
	<li>The game object hierarchy is not restored automatically. In situations where this is needed, the developer will need to take care of that himself, usually by implementing entities' OnAfterLoad methods.
Note that transform members like position can still be easily persisted by wrapping them into a property. Restoring the hierarchy could be as simple as attaching the entity's transform to a parent entity.</li>
	<li>Some serialization mechanisms can not support references between entities and other features like polymorphism. At the moment this is the case with the XmlEncoder.</li>
	<li>This will need to be considered. Special care still needs to be taken to ensure the value types of persistent fields and properties are serializable in the chosen serialization mechanism. Every serializer demands it's own rules and constraints for serializable types. This really can not be avoided.</li>
</ul>
<h2>Other Comments</h2>
<ul>
	<li>Recently another use case was discovered which, in hindsight, should have been considered. That is the case where game entities are defined in the scene and removed during run-time. An example of this would be pickups that vanish when the player touches them. This should be handled in the final version but is not a difficult problem.</li>
</ul>
<h1>Conclusion</h1>
The current system is usable for most of the use cases it is likely going to deal with. One exception is the use case for removed entities, which was not initially considered, but handling this should not be difficult so it will likely work in the final version. Overall the project seems on track, including my thesis which is slowly accumulating a nice amount of content.
