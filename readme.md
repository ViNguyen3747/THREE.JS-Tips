# Three.js Tips

## MONITORING

### 1. Monitor FPS

Install stats.js

```bash
npm install --save stats.js
```

add to Script.js

```javascript
import Stats from "stats.js";

const stats = new Stats();
stats.showPanel(0); // 0: fps, 1: ms, 2: mb, 3+: custom
document.body.appendChild(stats.dom);
```

In tick() fnc

```javascript
const tick = () => {
  stats.begin();

  // ...

  stats.end();
};
```

### 2. Renderer informations

```javascript
console.log(renderer.info);
```

### 3. [Dispose of things](https://threejs.org/docs/#manual/en/introduction/How-to-dispose-of-objects)

```javascript
scene.remove(cube);
cube.geometry.dispose();
cube.material.dispose();
```

## LIGHTS

### 4. Avoid using Lights

Avoid using Three.js Lights, Use baked lights or cheap lights [AmbienrLight](https://threejs.org/docs/#api/en/lights/AmbientLight), [DirectionalLight](https://threejs.org/docs/#api/en/lights/DirectionalLight)

### 5. Avoid adding or Removing Lights

When you add or remove light from the scene, all the materials supporting lights will have to be recompiled.

## SHADOWS

### 6. Avoid using Shadows, use BAKED shadows instead

### 7. Optimize shadow aps

Use [Camera Helper](https://threejs.org/docs/#api/en/helpers/CameraHelper) to see the area in order to reduce it to the smallest area possible

For Example:

```javascript
directionalLight.shadow.camera.top = 3;
directionalLight.shadow.camera.right = 6;
directionalLight.shadow.camera.left = -6;
directionalLight.shadow.camera.bottom = -3;
directionalLight.shadow.camera.far = 10;

const cameraHelper = new THREE.CameraHelper(directionalLight.shadow.camera);
scene.add(cameraHelper);

// Make sure to use the smallest possible resolution with descent result for the mapSize
directionalLight.shadow.mapSize.set(1024, 1024);
```

### 8. castShadow and receiveShadow wisely

Decide wether or not the obj should cast/receive shadow

```javascript
//if there's nothing surrounding the sphere, then it doesn't need to receive any shadow
sphere.castShadow = true;
sphere.receiveShadow = false;
//floor doesn't need to cast shadow since it's the floor :) and nothing under them
floor.castShadow = false;
floor.receiveShadow = true;
```

### 9. Deactivate shadow audo update

You can deactivate this auto-update and alert Three.js that the shadow maps needs update only when **necessary**:

```javascript
renderer.shadowMap.autoUpdate = false;
renderer.shadowMap.needsUpdate = true;
```

## TEXTURES

### 10. Resize textures

reduce the **resolution** to the minimum

### 11. Keep a power of 2 resolutions

very very important for **mipmaps**

### 12. try to compress image file

for .jpg and .png, [tinyPNG](https://tinypng.com/)

you can use [.basis](https://github.com/BinomialLLC/basis_universal), much better but harder to generate

## GEOMETRIES

### 13. DO NOT update vertices

In order to animate vertices, do it in vertex shader

### 14. Munutilize geometries

Have multiple [Meshes](https://threejs.org/docs/#api/en/objects/Mesh) using the same geometry shape
For example:

```javascript
const geometry = new THREE.BoxGeometry(0.5, 0.5, 0.5);

for (let i = 0; i < 50; i++) {
  const material = new THREE.MeshNormalMaterial();

  const mesh = new THREE.Mesh(geometry, material);
  mesh.position.x = (Math.random() - 0.5) * 10;
  mesh.position.y = (Math.random() - 0.5) * 10;
  mesh.position.z = (Math.random() - 0.5) * 10;
  mesh.rotation.y = (Math.random() - 0.5) * Math.PI * 2;
  mesh.rotation.z = (Math.random() - 0.5) * Math.PI * 2;

  scene.add(mesh);
}
```

### 15. Merge geometries to have only 1 draw call

If you cannot merge geometries because you need to have control over the meshes independently look at tip 18

```javascript
import * as BufferGeometryUtils from "three/examples/jsm/utils/BufferGeometryUtils.js";

// ...
const geometries = [];
for (let i = 0; i < 50; i++) {
  const geometry = new THREE.BoxGeometry(0.5, 0.5, 0.5);

  geometry.rotateX((Math.random() - 0.5) * Math.PI * 2);
  geometry.rotateY((Math.random() - 0.5) * Math.PI * 2);

  geometry.translate(
    (Math.random() - 0.5) * 10,
    (Math.random() - 0.5) * 10,
    (Math.random() - 0.5) * 10
  );

  geometries.push(geometry);
}

const mergedGeometry = BufferGeometryUtils.mergeBufferGeometries(geometries);
console.log(mergedGeometry);

const material = new THREE.MeshNormalMaterial();

const mesh = new THREE.Mesh(mergedGeometry, material);
scene.add(mesh);
```

## MATERIALS

### 16. Mutualize materials

```javascript
const geometry = new THREE.BoxGeometry(0.5, 0.5, 0.5);

const material = new THREE.MeshNormalMaterial();

for (let i = 0; i < 50; i++) {
  const mesh = new THREE.Mesh(geometry, material);
  mesh.position.x = (Math.random() - 0.5) * 10;
  mesh.position.y = (Math.random() - 0.5) * 10;
  mesh.position.z = (Math.random() - 0.5) * 10;
  mesh.rotation.x = (Math.random() - 0.5) * Math.PI * 2;
  mesh.rotation.y = (Math.random() - 0.5) * Math.PI * 2;

  scene.add(mesh);
}
```

### 17. use cheap materials

[MeshBasicMaterial](https://threejs.org/docs/#api/en/materials/MeshBasicMaterial), [MeshLamberMaterial](https://threejs.org/docs/#api/en/materials/MeshLambertMaterial), [MeshPhongMaterial](https://threejs.org/docs/#api/en/materials/MeshPhongMaterial)

## MESHES

### 18. InstancedMesh

cannot use tip 15 since you need to control every mesh independently

You create only one [InstancedMesh](https://threejs.org/docs/#api/en/objects/InstancedMesh), create a tranformation matrix ([Matrix4](https://threejs.org/docs/#api/en/math/Matrix4)) for each instance

```javascript
const geometry = new THREE.BoxGeometry(0.5, 0.5, 0.5);

const material = new THREE.MeshNormalMaterial();

const mesh = new THREE.InstancedMesh(geometry, material, 50);
scene.add(mesh);

for (let i = 0; i < 50; i++) {
  const position = new THREE.Vector3(
    (Math.random() - 0.5) * 10,
    (Math.random() - 0.5) * 10,
    (Math.random() - 0.5) * 10
  );

  const quaternion = new THREE.Quaternion();
  quaternion.setFromEuler(
    new THREE.Euler(
      (Math.random() - 0.5) * Math.PI * 2,
      (Math.random() - 0.5) * Math.PI * 2,
      0
    )
  );

  const matrix = new THREE.Matrix4();
  matrix.makeRotationFromQuaternion(quaternion);
  matrix.setPosition(position);

  mesh.setMatrixAt(i, matrix);
}

const tick = () => {
  //...

  //add this to change matrices after frames.
  mesh.instanceMatrix.setUsage(THREE.DynamicDrawUsage);
};
```

## MODELS

### 19. Low Poly

use low poly models. The fewer polygons, the better. To add details, use **normal maps**

### 20. Draco compression

If the model has a lot of details with very complex geometries, use the Draco compression. It can reduce weight drastically. The drawbacks are a potential freeze when uncompressing the geometry, and you also have to load the Draco libraries.

### 21. Gzip

Gzip is a compression happening on the server side. Most of the servers don't gzip files such as .glb, .gltf, .obj, etc.

## CAMERAS

### 22. Field of view

When objects are not in the field of view, they won't be rendered. That is called **frustum culling**.

reduce the camera's field of view. The fewer objects on the screen, the fewer triangles to render.

### 23. Near and far

you can reduce the **near** and **far** properties of the camera. If you have a vast world with mountains, trees, structures, etc., the user probably can't see those small houses out of sight far behind the mountains. Reduce the **far** to a decent value and those houses won't even try to be rendered.

## RENDERER

### 24. limit Pixel Ratio

```Javascript
renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))
```

### 25. Antialias

The default antialias is performant, but still, it's less performant than no antialias. Only add it if you have **visible aliasing** and **no performance issue**.

## POSTPROCESSING

### 26. Limit Passes

Each post-processing pass will take as many pixels as the render's resolution (including the pixel ratio) to render. If you have a **1920x1080** resolution with 4 passes and a pixel ratio of **2**, that makes **1920 _ 2 _ 1080 _ 2 _ 4 = 33 177 600** pixels to render. Be reasonable, and try to regroup your custom passes into one.

## Shaders

### 27. Specify the precision

```javascript
const shaderMaterial = new THREE.ShaderMaterial({
  precision: "lowp",
  // ...
});
```

### 28. Keep code simple in shader files

AVOID using **if** statement, use **clamp**

```javascript
modelPosition.y += clamp(elevation, 0.5, 1.0) * uDisplacementStrength;
```

Or as in the fragment shader, instead of these complex formulas for **r**, **g** and **b**:

```javascript
vec3 depthColor = vec3(1.0, 0.1, 0.1);
vec3 surfaceColor = vec3(0.1, 0.0, 0.5);
vec3 finalColor = mix(depthColor, surfaceColor, elevation);
```

### 29. use Textures

Perlin noise is bad for performance, unless you want to be able to animate it
Otherwise, you could use **texture2D()** and produce texture with tools like photoshop :)

### 30. Use Defines

if the values is not supposed to change, use **define** instead of **uniforms**

In vertex shader

```
#define uDisplacementStrength 1.5
```

Or in Script.js

```javascript
const shaderMaterial = new THREE.ShaderMaterial({

    // ...

    defines:
    {
        uDisplacementStrength: 1.5
    },

    // ...
}
```

### 31. DO CALCULATIONS in Vertex Shader as possible

then send the result to **Fragment shader**
