---
layout: post
title: Game Development Best Practices - GameSaves
date: 2019-12-19 22:16
author: sanderman0
comments: true
tags: [game development]
---
It recently occurred to me that I keep seeing the same kinds of patterns in many of the codebases get to work on, some of which I consider to be flaws, as they take problems that should be relatively simple and make them more difficult than they ought to be. These patterns then influence the code around them, sort like an infection. Once the rest of the codebase relies on flawed assumptions, it often becomes much more difficult and costly to fix the underlying root problem, as doing so may require extensive changes in other locations of the codebase. Though these issues could be prevented in early stages of a project with some thoughtful planning. That's why I decided to start a series on best practices in game development.

This post's topic is on systems for saving player progression on the local device. E.g. Systems handling save files on PC, or save slots on consoles, and cloud saving/syncing.
This post will *not* cover the case of online competitive games or MMOs that store all player progression data in a developer hosted database server.

## Example Problem

Here is a C# example of a save system API that I recently encountered inside a Unity game, with a condensed and simplified implementation.

```cs
public class SaveManager
{
    ...
    public bool FileExists(string fileName) {
        return File.Exists(GetPath(fileName));
    }

    public void Save(string fileName, SaveData data) {
        BinaryFormatter formatter = new BinaryFormatter();
        using (FileStream file = File.OpenWrite(GetPath(fileName))) {
            formatter.Serialize(file, data);
        }
    }

    public SaveData Load(string fileName) {
        BinaryFormatter formatter = new BinaryFormatter();
        using (FileStream file = File.OpenRead(GetPath(fileName))) {
            SaveData data = (SaveData)formatter.Deserialize(file);
            return data;
        }
    }
}
```
In short, a binary formatter is used for serialization, paired with .Net streams to interact with the filesystem.

This example looks simple and clean, but it hides a number of problems. These are not readily apparent when just writing to a file on PC, but they can become troublesome when considering other devices, or cloud platforms. I will cover these problems in the following sections and improve this example incrementally, where each step serves as a foundation for the next one. Errors and exceptions handling are important, but for this post I will focus on the happy path to keep the examples concise.

Sidenote: Using BinaryFormatter is not ideal as it is does an enormous amount of bookkeeping and as a result is slow and packs data inefficiently, but it's a good baseline as it doesn't require other dependencies besides what's available in .NET or Unity. Be aware that there are other serialization formats and libraries available out there, like eg. JSON, BSON, CBOR, ProtocolBuffers, MsgPack, SQLite. Those are outside the scope of this post.

## Separation of serialization and storage concerns
Separation of concerns is an important design principle in software engineering. Breaking problems into distinct parts allows us to easier manage inherent complexity.

- Serialization is the process of translating data structures or object state located throughout memory into a linear format suitable for storage. (typically a series of bytes, or text)
- Storage is writing this data to a storage medium, like an OS filesystem, or even to a remote server over a network.

The previous example has these two concerns interleaved in the same place, which makes it difficult to manage once functionality becomes more complex. (eg. due to adding error or exception handling, or threading) The following change fixes this and paves the way for the later improvements.

```cs
public class SaveManager {
    ...
    public bool FileExists(string fileName) {
        return storage.FileExists(fileName);
    }

    public void Save(string fileName, SaveData data) {
        BinaryFormatter formatter = new BinaryFormatter();
        MemoryStream stream = new MemoryStream();
        formatter.Serialize(stream, data);
        byte[] buffer = stream.ToArray();
        storage.Save(fileName, buffer);
    }

    public SaveData Load(string fileName) {
        byte[] buffer = storage.Load(fileName);
        MemoryStream stream = new MemoryStream(buffer);
        BinaryFormatter formatter = new BinaryFormatter();
        SaveData data = (SaveData)formatter.Deserialize(stream);
        return data;
    }
}

public class StorageBackend {
    public bool FileExists(string fileName) { return File.Exists(GetPath(fileName)); }
    public void Save(string fileName, byte[] buffer) { File.WriteAllBytes(GetPath(fileName), buffer); }
    public byte[] Load(string fileName) { return File.ReadAllBytes(GetPath(fileName)); }
}
```
Here we have kept the serialization logic inside `SaveManager` and delegated the responsibility of saving the raw data to a different object of type `StorageBackend`. This helps with readability.
- `GetPath` is no longer used in `SaveManager` as it no longer needs to care about that aspect. `StorageBackend` knows how and where to store the raw data.
- Similarly, the storage layer no longer needs to know anything about serialization.
- All file handling moved into `StorageBackend` where we can use simple file handling methods dealing directly with byte arrays, which is simpler than using a FileStream.
- In `SaveManager` we can eliminate the `using` blocks because unlike a FileStream a MemoryStream doesn't use any resources other than memory, so do not need to properly close and dispose it. (Though you might consider doing so to be a good habit anyway.)
- Other serialization methods besides BinaryFormatter often allow serializing to a byte array directly, which means you could remove the MemoryStream step as well.

It's worth noting that we might be using more memory here compared to the previous sample, as we are using an intermediate buffer we did not use before, rather than writing directly to disk. This is a trade-off between between memory usage, and code clarity and flexibility. For most games, this should not matter as their game save data is relatively small, while other games with large worlds (eg. Minecraft) can use chunking to mitigate memory issues if needed.

## Plugging storage implementations using interface

Sooner or later, you will want the ability to swap out the storage layer for some alternative implementation for various reasons:
- Testing
- Integrating things like cloud saves (allowing users to play on each of their devices without the worry and hassle of transferring saves between devices)
- Embedded save files (eg. To give users a specific experience during a public event or presentation for publisher)
- Building for consoles or mobile

Since most of the work to seperate storage functionality into a separate class was already done in the previous section, all we really need to do here is to define an interface or abstract class. (or whatever other mechanism your language offers for runtime polymorphism) Now we can easily define more storage backends with each their own implementation and we can inject whichever one we want into the SaveManager depending on the situation or platform. In this case I'm passing it into the `SaveManager` constructor.

```cs
public class SaveManager {
    SaveManager(Storage storage) { this.storage = storage; }
    ...
}

public interface Storage {
    bool FileExists(string fileName);
    void Save(string fileName, byte[] buffer);
    byte[] Load(string fileName);
}

public class MockupStorage : Storage { ... }
public class PcStorage : Storage { ... }
public class SteamStorage : Storage { ... }
public class XboxStorage : Storage { ... }
public class Ps4Storage : Storage { ... }
```

Now we can neatly contain all platform specific code inside their own wrapper. SaveManager will not need to know anything about how to save a file on a PC or to a cloud or a console.

## Atomicity
When we try to save player progress data to the filesystem or elsewhere, there is always a possiblity that this operation will fail in some way, for a myriad of reasons:
- The data structure being serialized for writing may contain fields that the serialization libary is unable to serialize. (e.g. cyclic graphs tend to be problematic)
- The data being loaded from a file might be malformed, causing deserialization to fail.
- The file system might throw exceptions due to denying access, or a full drive, or a failing one.
- The cloud server you're talking to might stop responding, or take a reaaaally long time. Or the network might drop your packets, or get cut with a backhoe.
- A stray cosmic ray might flip a bit and crash the operating system.

What happens if something goes wrong halfway during saving and we end up with a half-written file? The file will likely be malformed, and the user unpleasantly surprised to notice his last save cannot be loaded and his progress is gone. This is especially bad if an earlier save file was overwritten by a corrupted one. The player lost all progress, rather than just the progress since the previous save.

Obviously we **never ever** want to this to happen. Whenever we interact with storage, we want these *indivisible* and *irreducable* operations to either succeed completely, or to not occur at all. We do not want any weird in-between states. We want atomicity, like the transactions in ACID databases.

The iteration in the first section already fixed the serialization issue. Previously we would stream directly to disk during serialization. A failure in the serialization process would immediately produce a bad file. After the first iteration, we only start trying to write the file once we already have completed the entire serialization process.

That still leaves us with the process of writing the file itself. How can we make this more all-or-nothing? That depends on the back-end storage implementation. PC filesystems allow a write-and-move trick, while other systems like gaming consoles offer dedicated APIs for game state saving involving mounting/unmounting files or directories, making backups and remote syncing. Here's a way to improve the save operation on PC.
```cs
public class PcStorage {
    ...
    public void Save(string fileName, byte[] buffer) 
    {
        string path = GetPath(fileName);
        string tmpPath = GetTempPath(fileName);
        File.WriteAllBytes(tmpPath, buffer);
        File.Move(tmpPath, path);
    }
    ...
}
```
First we write to a temporary path. Once we are completely finished writing to that file, we rename it to what we really want. Renaming a file is a fast operation and extremely unlikely to be interrupted, so for practical purposes we can consider this an atomic operation.
If something goes wrong during write, we do not care about what happens to the temporary file. We can throw an exception (or other signal) and let the game and the user know that saving failed for some reason, and the user can try to save the current game state again after resolving whatever issue caused the filesystem error.

## Asynchronicity / Parallel Concurrency
Up until now, all of the method signatures in our SaveManager or Storage carry the assumption of being synchronous in nature. This API makes it close to impossible for storage implementations to use parallelism under the hood through threading.

The methods `FileExists` and `Load` can never be implemented in a truly asynchronous fashion, because they require a return value immediately. By extension, the caller and other methods higher up in the call stack also assume that these methods return a useful result immediately.
Best to fix this early before many important game systems are 'infected' with these assumptions and hard to untangle.

*(Technically, you could use a busy-loop/busy-wait to wait on another thread, but you'd still be blocking the main thread until the waiting method can return the result, so you're not gaining much.)*

There's a number of reasons why we want to allow and even encourage asynchronous processing using threading. Most of them boil down to the fact that blocking the Main Thread (or main game loop) is bad so we should try not to block the main thread whenever it can be avoided.

- Stalls and frame drops are generally bad user experience when not inside a loading screen. (eg. checkpoint saving during gameplay)
- During loading screens it is common to render a spinner or progress indicator, to inform the user we're busy and haven't crashed. This means we cannot block the main thread.
- Some systems require a specific function to be called regularly, to indicate to the system that the application has not frozen. (eg. due to a bug causing an infinite loop)

In addition to these factors, the storage implementation has more freedom in how it implements functionality under the hood if not constrained to a synchronous control flow. For example: It may need to communicate with some other system thread through some platform library. Or it may need to send a request to a remote webserver and wait for a response before it can provide a result. **A synchronous API can easily be wrapped inside an asynchronous API, but the reverse is not the case.**

There's various ways to make your methods asynchronous. Ultimately they all boil down to passing in some input, and then having some kind of way to get the result out later. The implementation can then do whatever it wants, like spawning threads, heavy calculations, waiting for stuff.. All without affecting the main thread.

You can explicitly pass in a response object or struct as parameter. The caller can read the response later to determine if the request was completed and if so access any result or error data. The response could be polled periodically, or some event handler could be called to indicate the response is ready. This sort of approach is quite common in APIs for consoles so it can be good to get familiar with it ahead of time:
```cs
public class LoadRequest { ... }
public class LoadResponse { ... }

public interface Storage {
    ...
    byte[] Load(LoadRequest request, LoadResponse response);
    ...
}
```

Another option is to pass in a callback function. Here we use a C# delegate to pass in a callback method to be used to return the result:
```cs
public interface Storage {
    ...
    byte[] Load(LoadRequest request, Action<LoadResponse> onDone);
    ...
}
```
This approach seems very friendly and easy to work with for callers at first glance, as there is supposedly no need to do polling. But there is a catch. That callback could be invoked from another thread which is not the main thread. If this was not expected, then the callback handler method might end up causing a threading warning from Unity if lucky, or end up in a data race with the main thread if unlucky.

If you're going to use threading, at least one side is going to have to do something to ensure that results are correctly passed back to the main thread. Make this responsibility explicit. For example, you could give each storage implementation an Update method, which the storage implementation can and should use to invoke any pending callbacks from the main thread.

Of course high level languages include features like async-await. These can also help to make asynchronous programming easier. Though try to be aware of how they work under the hood before using them. Async-await in C# is built upon the Task model. Within Unity, that means async-await tasks tend to run on the main thread, just like Unity's coroutines, unless you take measures to spawn a new thread.