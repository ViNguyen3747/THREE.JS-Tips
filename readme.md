# Three.js Tips

## Monitoring

### Monitor FPS

Install stats.js

```
npm install --save stats.js
```

add to Script.js

```
import Stats from 'stats.js'

const stats = new Stats()
stats.showPanel(0) // 0: fps, 1: ms, 2: mb, 3+: custom
document.body.appendChild(stats.dom)
```

In tick() fnc

```
const tick = () =>
{
    stats.begin()

    // ...

    stats.end()
}
```

### Renderer informations

```
console.log(renderer.info)
```
