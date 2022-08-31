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

### 6. AVoid using Shadows, use BAKED shadows instead

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
