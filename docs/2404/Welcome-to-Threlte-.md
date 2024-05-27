<!--yml
category: 未分类
date: 2024-05-27 13:37:25
-->

# Welcome to Threlte.

> 来源：[https://threlte.xyz/](https://threlte.xyz/)

Threlte comes with integrations for

[Rapier](https://rapier.rs/)

, a best-in-class physics engine,

[Theatre.js](https://www.theatrejs.com/)

, an animation library with a professional motion design toolset and

`@threlte/gltf`

, a tool to transform GLTF files into Threlte components.

On top of that

`@threlte/extras`

provides a collection of useful components and utilities to get you started.

 [Documentation](/docs/reference/extras/getting-started) 

[](/docs/reference/extras/getting-started)

@threlte/rapier

<astro-island uid="Qg4qo" component-url="/_astro/CopyCodeButton.9N1cNqvV.js" component-export="default" renderer-url="/_astro/client.bWR2MAd2.js" props="{&quot;code&quot;:[0,&quot;<script>\n  import { RigidBody, AutoColliders } from '@threlte/rapier'\n  import { T } from '@threlte/core'\n</script>\n\n<RigidBody>\n  <AutoColliders shape={'ball'}>\n    <T.Mesh receiveShadow castShadow>\n      <T.SphereGeometry args={[0.25]} />\n      <T.MeshStandardMaterial />\n    </T.Mesh>\n  </AutoColliders>\n</RigidBody>\n&quot;]}" ssr="" client="idle" opts="{&quot;name&quot;:&quot;CopyCodeButton&quot;,&quot;value&quot;:true}" await-children=""></astro-island>

```
<script>
 import { RigidBody, AutoColliders } from  '@threlte/rapier'
 import { T } from  '@threlte/core'
</script>
 <RigidBody>
 <AutoColliders  shape={'ball'}>
 <T.Mesh  receiveShadow  castShadow>
 <T.SphereGeometry  args={[0.25]} />
 <T.MeshStandardMaterial />
 </T.Mesh>
 </AutoColliders>
</RigidBody>
```

@threlte/extras

<astro-island uid="Z1GitkV" component-url="/_astro/CopyCodeButton.9N1cNqvV.js" component-export="default" renderer-url="/_astro/client.bWR2MAd2.js" props="{&quot;code&quot;:[0,&quot;<script>\n  import { GLTF, Float } from '@threlte/extras'\n</script>\n\n<Float>\n  <GLTF\n    castShadow\n    receiveShadow\n    url={'/models/flower.glb'}\n    position.y={1}\n    scale={3}\n  />\n</Float>\n&quot;]}" ssr="" client="idle" opts="{&quot;name&quot;:&quot;CopyCodeButton&quot;,&quot;value&quot;:true}" await-children=""></astro-island>

```
<script>
 import { GLTF, Float } from  '@threlte/extras'
</script>
 <Float>
 <GLTF
 castShadow
 receiveShadow
 url={'/models/flower.glb'}
 position.y={1}
 scale={3}
 />
</Float>
```