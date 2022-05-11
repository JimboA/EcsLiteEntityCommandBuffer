# EcsLiteEntityCommandBuffer
Entity command buffer for [LeoECS Lite](https://github.com/Leopotam/ecslite).
It will allow you to create delayed commands on entities (like Add/Set/Del components etc.) and execute them in special systems.
Why? I personally use it for turn base games, but you can do it your way.

> Tested on unity 2020.3 (not dependent on it) and contains assembly definition for compiling to separate assembly file for performance reason.

> **Important!** Don't forget to use `DEBUG` builds for development and `RELEASE` builds in production: all internal error checks / exception throwing works only in `DEBUG` builds and eleminated for performance reasons in `RELEASE`.

# Installation

## As unity module
This repository can be installed as unity module directly from git url. In this way new line should be added to `Packages/manifest.json`:
```
"com.jimboa.ecsliteentitycommandbuffer": "https://github.com/JimboA/EcsLiteEntityCommandBuffer.git",
```

## As source
If you can't / don't want to use unity modules, code can be downloaded as sources archive from `Releases` page.

# How to use
To start using the command buffer you must create a special ECB system. Just inherit your system from the EcbSystem class:
```csharp
using Leopotam.EcsLite;
using JimboA.EcsLite.ECB;

class UserEcbSystem : EcbSystem, IEcsRunSystem
{   
    public void Run (EcsSystems systems) 
    {
        // Will be called on each EcsSystems.Run() call.
    }
}
```
And add it to systems in startup:
```csharp
...
EcsSystems systems = new EcsSystems (world);
systems
    .Add (new UserEcbSystem(world))
    .Init ();
...
```
The ECB system is responsible for creating and executing the buffer. Think of it as a synchronization point. For each such system, its own buffer is created and the world is attached to it. Each command buffer can only be associated with one world and one ECB system.

The command buffer can be accessed from anywhere in your code via the ecs world:
```csharp
using Leopotam.EcsLite;
using JimboA.EcsLite.ECB;

class UserRunSystem : IEcsRunSystem
{   
    public void Run (EcsSystems systems) 
    {
        ...
        var world = systems.GetWorld(); // getting the default world
        var ecb = world.GetCommandBufferFrom<UserEcbSystem>(); // getting buffer from your EcbSystem
        ref var laterComponent = ref ecb.Add<UserComponent>(entity, out var cmdEntity) // when buffer will be executed a UserComponent will be added to entity
        // out parameter here return a packed ecb command. You can store it and execute that particular command later. Or just ignore it with out _
        ...
    }
}
```
In addition to the usual commands, you can also save specific sequences and execute them later:
```csharp
class UserRunSystem : IEcsRunSystem
{   
    public void Run (EcsSystems systems) 
    {
        ...
        var world = systems.GetWorld(); // getting the default world
        var ecb = world.GetCommandBufferFrom<UserEcbSystem>(); // getting buffer from your EcbSystem
        var seq = ecb.Sequence(out var seqEntity); // getting special sequence
        seq.Add<UserComponent>(entity1);
        seq.Add<UserSecondComponent>(entity1);
        seq.Del<AnotherComponent>(entity2);
        //you can store seqEntity and execute that sequence later.
        ...
    }
}
```
Now let's talk about execution.
Commands are executed in ecb systems. 

> **Important!** EcbSystems only provides the API for execution. The order and rules of execution are determined by the user.

For instance:
```csharp
using Leopotam.EcsLite;
using JimboA.EcsLite.ECB;

class UserEcbSystem : EcbSystem, IEcsRunSystem
{   
    private EcsFilter _userFilter;
    ...
    public void Run (EcsSystems systems) 
    {
        foreach(var entity in _userFilter)
        {
            ExecuteCommandsOnEntity(entity); // execute all commands that belong to the entity
        }    
    }
    ...
}
```
All execution methods has summary, so just read it and use :)

> **Important!** The command buffer is deterministic. In what order the commands were assigned in which order they will be executed if the user has not specified otherwise.

# License
The software is released under the terms of the [MIT license](./LICENSE.md).

No personal support or any guarantees.
