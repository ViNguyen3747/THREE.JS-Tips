# Three.js Tips

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

### 4. Avoid using Lights

Avoid using Three.js Lights, Use baked lights or cheap lights [AmbienrLight](https://threejs.org/docs/#api/en/lights/AmbientLight), [DirectionalLight](https://threejs.org/docs/#api/en/lights/DirectionalLight)

### 5. Avoid adding or Removing Lights

When you add or remove light from the scene, all the materials supporting lights will have to be recompiled.

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

### 10. Resize textures

reduce the **resolution** to the minimum

### 11. Keep a power of 2 resolutions

very very important for **mipmaps**

### 12. try to compress image file

for .jpg and .png, [tinyPNG](https://tinypng.com/)

you can use [.basis](https://github.com/BinomialLLC/basis_universal), much better but harder to generate

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
```
