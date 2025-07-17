---
layout: post
title:  "Unity Dots Learning(1) Basic Concepts And Project Setup"
date:   2025-07-17 10:00:00 +0800
categories: Unity
tags: Blog
---

## Package Required
Entities
![Entities](/assets/images/2025-07-17-17-57-32.png)
Entities Graphics
![Entities Graphics](/assets/images/2025-07-17-17-57-52.png)
Unity Physics
![Unity Physics](/assets/images/2025-07-17-17-58-23.png)
## ECS Basic Concepts
### Entity
The entity in unity dots basically is an unique id, which can be used to access to its components.
### Component
The components are used for storing data, and we shouldn't put any logic into them. They are usually struct values.
* A component without any fields is called a tag component. It can be useful for queries.
* The DynamicBuffer is a component type that can used as a resizable array. They are structs that implement the IBufferElementData interface.
### System
We should put most of our game logic in systems. They update every logic frame.
### Archetype and Chunk
![](/assets/images/2025-07-17-18-45-17.png)
Archetype represent a particular unique combination of component types in a world. All entites with component A and B are stored in an archetype, and all entites only with component A are stored in another archetype.

Entites and their components are stored in chunks in the archetype. The size of a chunk is 16kb, and one chunk can store up to 128 entites. This number can be lower with bigger entites.

The entity's ID and their components of each type are stored in separate arrays in the chunk. For example, in the image, the first array is for entity IDs, the second array is for component A, and the third array is for component B. The entity id and components stored at index 0 of the 3 arrays can make up the first entity, and all the datas stored at index 1 can make up the second entity, and so on.
## Why Is ECS Good
* The components are usually struct, and they are stored tightly in arrays. As a result, it can reduce the cache miss a lot, which can make everything much faster.
* In traditional unity developing, every component on a gameobject is a monobehaviour, and monobehaviour is really heavy. In ECS, the components are much lighter. The structs are on the unmanaged memory, and they won't trigger GC.

## Bake And SubScene
We can create subscene in the unity editor, and they are very useful in ecs.
![](/assets/images/2025-07-17-19-16-33.png)

From what I have learned so far, the traditional unity gameobjects under subscenes can be baked into ECS entites upon running.

![](/assets/images/2025-07-17-19-20-58.png)

The gameobject cube is under the entites subscene, and all the original unity components: Transform, Collider, Mesh renderer ..... are converted into ECS components. It's really convenient for us to setup our scene in Unity Editor. We can also bake our own monobehaviour game components into ECS components if we implment a coresspond baker class and its bake function.

Create some cubes and a plane under the subscene, add rigidbody component to them, we can see that they just work fine in ECS, just like they did originally.

## References
unity official ECS document:
https://docs.google.com/document/d/1R6E4IDpfLatwHITlCND0i5TuMVG0CMGsentFL-3RQT0/edit?tab=t.0