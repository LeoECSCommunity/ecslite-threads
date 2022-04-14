# Threads support for LeoEcsLite C# Entity Component System framework
Threads support for [LeoECS Lite](https://github.com/LeoECSCommunity/ecslite).

> Tested on unity 2020.3 (not dependent on it) and contains assembly definition for compiling to separate assembly file for performance reason.

> **Important!** Don't forget to use `DEBUG` builds for development and `RELEASE` builds in production: all internal error checks / exception throwing works only in `DEBUG` builds and eleminated for performance reasons in `RELEASE`.

# Table of content
* [Socials](#socials)
* [Differences to Leo Red](#differences-to-leo-red)
* [Installation](#installation)
    * [As unity module](#as-unity-module)
    * [As source](#as-source)
* [Example](#example)
    * [Component](#component)
    * [ThreadSystem](#threadsystem)
    * [Thread](#thread)
* [License](#license)

# Socials
[![discord](https://img.shields.io/discord/963730852452388894.svg?label=New%20Community%20Discord%20server&style=for-the-badge&logo=discord)](https://discord.gg/ZAhCUv5YQt)

[![discord](https://img.shields.io/discord/404358247621853185.svg?label=Old%20Leo%20Discord%20server&style=for-the-badge&logo=discord)](https://discord.gg/5GZVde6)

# Differences to Leo Red
This is a community maintained fork of LeoECS from the point at which its license (and some obsolete code) has changed.  
This fork is **not** maintained by Leopotam.  
LeoECSCommunity repositories will always be MIT licensed, and will be backwards compatible with the last LeoECS MIT version (for the time being).  

If you have any issues with this fork, please address them here in LeoECSCommunity repositories, or our Discord sever.  
But we do accept PR's and issues from contributors from Leo Red as well.

# Installation

## As unity module
This repository can be installed as unity module directly from git url. In this way new line should be added to `Packages/manifest.json`:
```
"com.leoecscommunity.ecslite.threads": "https://github.com/LeoECSCommunity/ecslite-threads.git",
```
By default last released version will be used. If you need trunk / developing version then `develop` name of branch should be added after hash:
```
"com.leoecscommunity.ecslite.threads": "https://github.com/LeoECSCommunity/ecslite-threads.git#develop",
```

## As source
If you can't / don't want to use unity modules, code can be downloaded as sources archive of required release from [Releases page](`https://github.com/LeoECSCommunity/ecslite-threads-unity/releases`).

# Example

## Component
```csharp
struct C1 {
    public int Id;
}
```
## ThreadSystem
```csharp
class TestThreadSystem : EcsThreadSystem<TestThread, C1> {
    protected override int GetChunkSize (EcsSystems systems) {
        // How many entities will be processed in one chunk.
        // This method will be called each frame.
        return 1000;
    }

    protected override EcsWorld GetWorld (EcsSystems systems) {
        // World for component pools and filter.
        return systems.GetWorld ();
    }
    
    protected override EcsFilter GetFilter (EcsWorld world) {
        // Filter for processing.
        return world.Filter<C1> ().End ();
    }

    // Optional additional initialization of job structure.
    protected override void SetData (EcsSystems systems, ref TestThread thread) {
        thread.DeltaTime = Time.deltaTime;
    }
}
```
> **Important!** EcsThreadSystem supports up to 4 component pools.
 
## Thread
```csharp
struct TestThread : IEcsThread<C1> {
    public float DeltaTime;
    int[] _entities;
    C1[] _pool1;
    int[] _indices1;

    public void Init (int[] entities, C1[] pool1, int[] indices1) {
        // entities array.
        _entities = entities;
        // dense array of components.
        _pool1 = pool1;
        // sparse indices array of components.
        _indices1 = indices1;
    }

    public void Execute (int fromIndex, int beforeIndex) {
        for (int i = fromIndex; i < beforeIndex; i++) {
            var e = _entities[i];
            ref var c1 = ref _pool1[_indices1[e]];
            c1.Id = (c1.Id + 1) % 10000;
        }
    }
}
```

# License
The software is released under the terms of the [MIT license](./LICENSE.md).

No personal support or any guarantees.
