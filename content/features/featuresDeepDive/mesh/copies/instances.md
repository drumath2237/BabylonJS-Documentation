---
title: Instances
image:
description: Learn all about the instancing system in Babylon.js.
keywords: diving deeper, meshes, mesh transformation, transformation, instancing
further-reading:
  - title: How To Use Thin Instances
    url: /features/featuresDeepDive/mesh/copies/thinInstances
video-overview:
video-content:
  - title: Fun with Instance Buffers
    url: https://youtu.be/rlODXrsdseA
---

## How to use Instances

Instances are an excellent way to use hardware accelerated rendering to draw a huge number of identical meshes (let's imagine a forest or an army).

Instances are built from a mesh with the following code:

```javascript
// In this case we're loading our mesh from an external source.
BABYLON.ImportMeshAsync("https://www.babylonjs.com/assets/Tree/tree.babylon", scene).then(function (result) {
  var mesh = result.meshes[0];
  // Make the "root" mesh not visible. The instanced versions of it that we
  // create below will be visible.
  mesh.isVisible = false;
  for (let index = 0; index < 100; index++) {
    var newInstance = mesh.createInstance("i" + index);
    // Here you could change the properties of your individual instance,
    // for example to form a diagonal line of instances:
    //  newInstance.position.x = index;
    //  newInstance.position.z = index;
    // See below for more details on what can be changed.
  }
});
```

A mesh can have as many instances as you want.

Each instance has the same material as the root mesh. They can vary on the following properties:

- `position`
- `rotation`
- `rotationQuaternion`
- `setPivotMatrix`
- `scaling`

Note: related are **thin instances**, if you want yet more performances but with less control on each instance. See the [dedicated page](/features/featuresDeepDive/mesh/copies/thinInstances) for further information.

## Use Instances with Node Material

The [Node Material](/features/featuresDeepDive/materials/node_material) is a powerful tool that allows creating shaders without having to write [GLSL](https://www.khronos.org/opengl/wiki/OpenGL_Shading_Language). To use a Node Material in a mesh with Instances, you need to add the Instances node to it so that the object is properly instanced:

![Use Instances with Node Material](/img/how_to/instances-node.png)

The Instances node also gives information on the instance number of the object, which can be useful to, for example, color each instance differently.

<Playground id="#PB1NS6#6" title="Using Node Material with Instances" description="Use Instances node on the Node Material"/>

## Instancing a glTF Object

When you instantiate a glTF object, you need to make sure that the new instance will be under the same parent or you need to remove the parent from the source object.

This is because every gltf file comes from a right handed world. To get it into Babylon.js left handed world, we are adding an arbitrary parent that is adding a negative scale on z.

So when instancing a glTF object you have to (either):

- Call `source.setParent(null)`
- Or call `newInstance.setParent(source.parent)`

## Custom Buffers

You also have the opportunity to specify per instance values for any attribute. For instance (no pun intended), if you want to have a specific color per instance, you only need to provide a vertex buffer flagged as "instanceable" and fill it with a color per instance:

```javascript
let instanceCount = 1000;
let colorData = new Float32Array(4 * instanceCount);

for (let index = 0; index < instanceCount; index++) {
  colorData[index * 4] = Math.random();
  colorData[index * 4 + 1] = Math.random();
  colorData[index * 4 + 2] = Math.random();
  colorData[index * 4 + 3] = 1.0;
}

var buffer = new BABYLON.VertexBuffer(engine, colorData, BABYLON.VertexBuffer.ColorKind, false, false, 4, true);
box.setVerticesBuffer(buffer);
```

The last parameter of the VertexBuffer constructor is the one to set to true to flag it as instanceable.

Example: <Playground id="#8L50Q3#203" title="Custom Buffers Example 1" description="Simple example of custom buffers."/>

The other way is to register a custom buffer with `registerInstancedBuffer`:

```javascript
mesh.registerInstancedBuffer("color", 4); // 4 is the stride size eg. 4 floats here
```

Using this API, you can specify a new buffer that will be instantiated. This means that each instance will have its own value. You can specify this value on the root mesh and on every single instance:

```javascript
box.instancedBuffers.color = new BABYLON.Color4(Math.random(), Math.random(), Math.random(), 1);
let instance = box.createInstance("box1");
instance.instancedBuffers.color = new BABYLON.Color4(Math.random(), Math.random(), Math.random(), 1);
```

The system will take care of updating the internal vertex buffer.

Example: <Playground id="#YPABS1#183" title="Custom Buffers Example 2" description="Simple example of custom buffers."/>

## Using Custom Buffers with Node Material

If you want to use custom buffers in conjunction with node materials, you can access the instanced buffers color with the mesh.color block. Make sure you have added the Instances block to your graph and then the mesh.color block will reference the instanced buffers color assigned to the instance.

![Use Instanced Buffers Color with Node Material](/img/how_to/instances-node-meshColor.png)

Example: <Playground id="#D6GB23" title="Custom Buffers in Node Material" description="Using custom buffers to drive texture offset in node material."/>

There is a great article from Simon Trushkin, a Senior Technical Artist working at ClickON3D, on [using custom buffers with custom blocks in node material](https://www.linkedin.com/pulse/advanced-use-instance-buffers-node-materials-clickon-web3d-nmdme/). This technique is all about optimization and demonstrating how combining instances, custom buffers, and node material can result in a huge boost to the frame rate of the scene.

## Advanced Control

You can decide to control the world matrix instanced buffer the same way you control the custom buffers.

To do so, just run the following code:

```javascript
mesh.manualUpdateOfWorldMatrixInstancedBuffer = true;
```

When this mode is activated, you can update the world matrix instanced buffer with this code:

```javascript
mesh.worldMatrixInstancedBuffer.set(mat, offset); // mat is the matrix you want to store at the given offset
offset += 16; (a matrix is composed of 16 floats
```

It is recommended to freeze the active meshes when controling the world matrix instanced buffer to avoid having a discrepancy between the values you store and the number of active instances:

```javascript
scene.freezeActiveMeshes(true);
```

You can find a complete example here: <Playground id="#HJGC2G" title="Instancing Advanced Control" description="Simple example of instancing advanced controls."/>

## Support

Instances are supported for collisions, picking, rendering and shadows. Even if the current hardware does not support hardware accelerated instances, babylon.js will be able to optimize rendering to take instances into account.

Starting from 5.0, instances that have a transparent material applied can be sorted from back to front to remove/limit rendering artifacts. This mode is enabled by setting `BABYLON.Mesh.INSTANCEDMESH_SORT_TRANSPARENT = true`.

Note that for performance sake the master mesh (the mesh from which instances are created by calling `masterMesh.createInstance`) is not taken into account in the sorting process. So, to avoid rendering artifacts between this mesh and its instances, this mesh should be disabled (`masterMesh.setEnabled(false)`).

## Using 3D Modeler to Create Instances

## Blender

Using Blender, you can create instances of a mesh by just creating a linked object:

![](/img/how_to/use-instance/blender-linked-object.jpg)

## 3ds Max

Using 3DS Max, you can create instances of a mesh by just creating a clone instance object with clic right on the object:

![](/img/how_to/use-instance/3ds-linked-object.jpg)

## Limitations

- You can use instances with LOD but one limitation will apply in this case: You will have to hide the root objects.
  Here is an example where LODs reuse instances:
  <Playground id="#0720FC#10" title="Instances and LODs" description="Simple example of instancing and LODs."/>

- Instances with a world matrix where determinant is different than root mesh world matrix will be rendered separately (like a regular mesh). This mostly happens when the sign of the scaling vector is different between an instance and the root mesh.

- When using motion blur, the engine needs to store world matrices of the previous frame to compute velocity. Usually, this part is taken care of internally, but in certain cases you may have to specify these matrices manually. You may in fact see weird blurring artifacts if you update your world matrix buffer manually (using `mesh.manualUpdateOfWorldMatrixInstancedBuffer = true;`). In that case, to also update previous world matrices, you must enable the corresponding flag :

```javascript
mesh.manualUpdateOfPreviousWorldMatrixInstancedBuffer = true;
```

And similarly to world matrices, update the previous world matrices :

```javascript
mesh.previousWorldMatrixInstancedBuffer.set(previousMat, offset);
```

Here is an example of manual update of world matrices along with previous world matrices, to use motion blur correctly with instances :
<Playground id="#HJGC2G#58" title="Instances previous matrices motion blur" description="Updating manually previous world matrices for instances to work with motion blur"/>

## Demos

- Trees: <Playground id="#YB006J#75" title="Instancing Trees Example" description="Simple example of instancing with trees."/>
- 10,000 Icospheres: <Playground id="#c2ynt9#12" title="10,000 Icospheres" description="Simple example of instancing with 10,000 icospheres."/>
