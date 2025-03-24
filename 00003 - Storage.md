# Storage

## Current Inheritance from Smalltalk-80

Psyche has inherited the Smalltalk-80 storage approach from Self. The fundamental concept is that objects are "alive," and the full object graph is serialized to disk on command rather than automatically.

This approach has two major downsides:

1. **Memory Restrictions**: You're restricted to the size of the object graph available in memory. This isn't as problematic for Squeak, which is 64-bit clean, but for Self (which is 32-bit only), there's a very low ceiling on the number of objects and the size of working memory.

2. **Scalability Issues**: Even with a 64-bit clean system, we cannot easily persist the entire state of the system. If we imagine a standard desktop or laptop running a single Self image, the image would have to encompass essentially the size of the hard drive, not just the working memory. This would include all videos, transcripts, game assets, etc. This approach clearly isn't feasible.

### Current Limitations

Beyond the snapshot mechanism, Self doesn't really have a comprehensive storage story. It can write and read files, leveraging the underlying Unix storage capabilities, but this creates a bifurcation between the Self environment in which you're working and the underlying storage you read from and write to. The Self world essentially becomes a program dealing with data outside itself, whereas we want code and data bound together as objects in the object graph.

## Goals

At minimum, we want to be able to encompass data that would typically reside on a local system. The question is how to design this storage approach without sacrificing the object orientation of the system.

The ideal storage solution would maintain the "everything is a live object" approach and simply make it work. If you wanted to look at a video file, you would fetch it into your system as a byte vector, then manipulate it or break it into separate objects (metadata, frames, etc.), all of which would persist.

In this model, "saving the image" would no longer be a concept—the image would simply be persistent. You might want checkpointing to roll back accidental destructive changes, but the basic premise is persistence.

However, this won't work without significant research. At minimum, you'd need something like an OOZE style system (but that ran on much, much more constrained hardware.) Non-volitile memory would help, alas for Optane, but even then, the lack of internal barriers between objects would cause issues.

It's not part of the Psyche plan to create hard internal barriers inside a running Self world. The barriers are between Self worlds; inside a world, everything is open. You can't have mutually hostile people collaborating in the same Self image because they can do anything, and we want them to be able to do anything. For untrusted software or users, you place them in different worlds that can communicate with each other.

## Data safety as an obvious goal

We need to protect against data loss. If you've downloaded your doctoral thesis into Self and it's the only copy, we don't want to lose it if you accidentally crash your world. We also don't want to roll back to an earlier state and lose recent changes.

### Easy win - offloading byteVectors

A simple approach might be "offloaded byte vectors" where we save byte vectors to disk but not the object relationships surrounding them. This would help for certain use cases, like a music player library where you only need to load the files when playing them. It would not help for persisting large interconnected objects graphs.

## Thoughts about Vats

One concept is a "vat" (for want of a better term) — a self-contained subset of objects with a gateway or root. You only interact with the vat through this gateway. If you crash your image, the vat remains intact on disk.

For larger storage needs, we could create a storage vat — a segregated object graph where communication happens through a translator and gatekeeper, returning translated versions of objects inside the graph.

The vat approach might help harmonize compute and storage, with the vat becoming responsible for storing and loading data. Compute (functional changes to the object graph) happens inside the vat.
