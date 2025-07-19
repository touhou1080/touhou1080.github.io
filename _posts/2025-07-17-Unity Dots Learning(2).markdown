---
layout: post
title:  "Unity Dots Learning(1) Basic System Example"
date:   2025-07-19 10:00:00 +0800
categories: Unity
tags: Blog
---

## SystemBase And ISystem

### SystemBase:
* Class
* Slower than ISystem

```
public partial class MoveSystemManaged: SystemBase
{
    public MoveSystemManaged() { }

    protected override void OnUpdate()
    {
        throw new System.NotImplementedException();
    }
}
```

### ISystem:
* Struct
* Better performance
* We should use this

```
partial struct MoveSystem : ISystem
{
    [BurstCompile]
    public void OnCreate(ref SystemState state)
    {
        
    }

    [BurstCompile]
    public void OnUpdate(ref SystemState state)
    {
        
    }

    [BurstCompile]
    public void OnDestroy(ref SystemState state)
    {
        
    }
}
```

## SystemState
SystemState is the parameter that is passed in OnCreate(), OnUpdate() and OnDestroy() function. According to the official document, it represents the state of the "system instance" and can be used to access the system's world, the entity manager, to create EntityQuery and so on. In the code of SystemState, we can see the comment of its initialization method: "Unmanaged systems call this function to initialize their backing system." From this we can infer that the term "system instance" likely means that for each system we create, Unity creates a corresponding SystemState object.

![](/assets/images/2025-07-19-04-44-38.png)

## Simple MoveSystem Example
```
using Unity.Burst;
using Unity.Entities;
using Unity.Mathematics;
using Unity.Transforms;

partial struct MoveSystem : ISystem
{
    [BurstCompile]
    public void OnCreate(ref SystemState state)
    {
        
    }

    [BurstCompile]
    public void OnUpdate(ref SystemState state)
    {
        float deltaTime = SystemAPI.Time.DeltaTime;
        foreach ((RefRW<LocalTransform> transform,RefRO<MoveSpeed> speed) in SystemAPI.Query<RefRW<LocalTransform>, RefRO<MoveSpeed>>())
        {
            transform.ValueRW.Position += new float3(speed.ValueRO.value * deltaTime, 0, 0);
        }
    } 

    [BurstCompile]
    public void OnDestroy(ref SystemState state)
    {
        
    }
}
```

This simple system can make the gameObject with MoveSpeed component move.

### Component Example And Authoring Component
```
public class MoveSpeedAuthoring : MonoBehaviour
{
    public float value;

    public class Baker: Baker<MoveSpeedAuthoring> {
        public override void Bake(MoveSpeedAuthoring authoring)
        {
            var entity = GetEntity(TransformUsageFlags.Dynamic);
            AddComponent(entity, new MoveSpeed
            {
                value = authoring.value
            });
        }
    }
}
```

```
public struct MoveSpeed : IComponentData
{
    public float value;
}

```
Authoring Component is the monoBehaviour component that can be baked into ECS component. In the above code, by implementing Baker class and bake method, we can make MoveSpeedAuthoring transformed into MoveSpeed under a ECS subScene.

It's very helpful considering the visualization. For example, we want to add the MoveSpeed to a cube in our scene, we can just drag the MoveSpeedAuthoring component like an ordinary monoBehaviour to the gameObject in the editor inspector.

### How Systems Are Initialized
It's automatic. Unity ECS automatically discovers systems in our project and instantiates them. We don't need to register it or perform any additional steps.

### Partial
According to the official update log(
https://docs.unity3d.com/Packages/com.unity.entities@0.50/manual/upgrade-guide.html#partial):

**Entities' internal code generation mechanism has changed from IL post-processing to Source Generators. This means you can see the underlying code that the ECS framework generates, and debug it accordingly. You can see this code in your project's /Temp/GeneratedCode folder. Because of this, the generated code must be valid C# and the partial keyword is needed to augment system types.**

### LocalTransform
It's like transform in ECS. According to the official document: Position, rotation and scale of this entity, relative to the parent, or in world space, if no parent exists. Like original transform, it has Position, Rotation and Scale fields.

### RefRW
Structs are value types, so when we get, for example, the LocalTransform component of an entity, we just get a copy. It means that even if we modify the copy extensively, the original LocalTransform of the entity won't be affected. So we need RefRW to get the reference to a struct.

### RefRO
It's short for Reference Read-Only. We can get the read-only reference of a component through it. It has better performance that getting the whole copy of a component, and makes the code clearer.

### DeltaTime
Use SystemAPI.Time.DeltaTime to get the deltatime of current logic frame.

### Query
Use SystemAPI.Query to get specific components. It supports querying multiple components, up to 7 at a time.
```
foreach (var (transform, moveSpeed) in SystemAPI.Query<RefRW<LocalTransform>, RefRO<MoveSpeed>>())
{
    transform.ValueRW.Position += new float3(moveSpeed.ValueRO.value, 0, 0) * SystemAPI.Time.DeltaTime;
}
```
