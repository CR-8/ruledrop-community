Build immersive, performant 3D web experiences using Three.js, WebGL, and related libraries. Use this skill when creating 3D scenes, interactive product viewers, particle systems, shader effects, 3D data visualizations, or any web experience that goes beyond flat 2D.

## Core Stack

- **Three.js** — primary 3D library for most use cases
- **React Three Fiber (R3F)** — Three.js in React via declarative JSX
- **@react-three/drei** — helpers: `OrbitControls`, `Environment`, `Text3D`, `useGLTF`, `MeshReflectorMaterial`
- **@react-three/postprocessing** — bloom, depth of field, chromatic aberration, SSAO
- **Leva** — dev controls for tweaking scene parameters
- **GSAP** — timeline-based animation, scroll-triggered 3D transitions
- **Cannon.js / Rapier** — physics simulation

## Scene Setup

Always start with a proper scene foundation:

```jsx
import { Canvas } from '@react-three/fiber'
import { Environment, OrbitControls, Stats } from '@react-three/drei'

function Scene() {
  return (
    <Canvas
      camera={{ position: [0, 2, 8], fov: 45 }}
      gl={{ antialias: true, toneMapping: THREE.ACESFilmicToneMapping }}
      shadows
    >
      <Environment preset="city" />
      <ambientLight intensity={0.3} />
      <directionalLight position={[5, 10, 5]} intensity={1.5} castShadow />
      <OrbitControls enableDamping dampingFactor={0.05} />
      <YourModel />
    </Canvas>
  )
}
```

## Materials & Shading

### PBR Materials (physically based)
```jsx
<meshStandardMaterial
  color="#e8d5b7"
  roughness={0.4}
  metalness={0.1}
  envMapIntensity={1.2}
/>
```

### Custom GLSL Shaders
Write shaders when you need effects that standard materials can't achieve:

```jsx
const vertexShader = `
  varying vec2 vUv;
  varying vec3 vNormal;
  void main() {
    vUv = uv;
    vNormal = normalize(normalMatrix * normal);
    gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
  }
`

const fragmentShader = `
  uniform float uTime;
  uniform vec3 uColor;
  varying vec2 vUv;
  void main() {
    float wave = sin(vUv.x * 10.0 + uTime) * 0.5 + 0.5;
    gl_FragColor = vec4(uColor * wave, 1.0);
  }
`

<shaderMaterial
  vertexShader={vertexShader}
  fragmentShader={fragmentShader}
  uniforms={{ uTime: { value: 0 }, uColor: { value: new THREE.Color('#6c63ff') } }}
/>
```

## Animation Patterns

### useFrame loop
```jsx
import { useFrame } from '@react-three/fiber'
import { useRef } from 'react'

function RotatingMesh() {
  const ref = useRef()
  useFrame((state, delta) => {
    ref.current.rotation.y += delta * 0.5
    // Use state.clock.elapsedTime for time-based effects
  })
  return <mesh ref={ref}><boxGeometry /><meshStandardMaterial /></mesh>
}
```

### Scroll-driven 3D with GSAP ScrollTrigger
```jsx
useEffect(() => {
  gsap.to(meshRef.current.rotation, {
    y: Math.PI * 2,
    scrollTrigger: {
      trigger: '#section',
      start: 'top center',
      end: 'bottom center',
      scrub: 1,
    }
  })
}, [])
```

## Particle Systems

```jsx
function Particles({ count = 5000 }) {
  const positions = useMemo(() => {
    const arr = new Float32Array(count * 3)
    for (let i = 0; i < count; i++) {
      arr[i * 3]     = (Math.random() - 0.5) * 20
      arr[i * 3 + 1] = (Math.random() - 0.5) * 20
      arr[i * 3 + 2] = (Math.random() - 0.5) * 20
    }
    return arr
  }, [count])

  return (
    <points>
      <bufferGeometry>
        <bufferAttribute attach="attributes-position" args={[positions, 3]} />
      </bufferGeometry>
      <pointsMaterial size={0.02} color="#ffffff" sizeAttenuation />
    </points>
  )
}
```

## Post-Processing Effects

```jsx
import { EffectComposer, Bloom, ChromaticAberration, DepthOfField } from '@react-three/postprocessing'

<EffectComposer>
  <Bloom luminanceThreshold={0.9} intensity={1.5} mipmapBlur />
  <ChromaticAberration offset={[0.002, 0.002]} />
  <DepthOfField focusDistance={0.01} focalLength={0.02} bokehScale={3} />
</EffectComposer>
```

## Performance Rules

- **Instancing** — use `InstancedMesh` for 100+ identical objects, never individual meshes
- **Geometry disposal** — call `geometry.dispose()` and `material.dispose()` on unmount
- **Texture compression** — use KTX2/Basis compressed textures for large assets
- **LOD** — swap high/low poly models based on camera distance
- **Draw calls** — merge static geometries with `BufferGeometryUtils.mergeGeometries()`
- **Target 60fps** — profile with `Stats` component, budget ~16ms per frame
- **Suspend heavy assets** — wrap model loaders in `<Suspense>` with a fallback

## Loading 3D Models

```jsx
import { useGLTF } from '@react-three/drei'

function Model({ url }) {
  const { scene } = useGLTF(url)
  return <primitive object={scene} />
}

// Preload outside component
useGLTF.preload('/model.glb')
```

Prefer `.glb` (binary GLTF) over `.gltf`. Compress with `gltf-transform` before shipping.

## Lighting Setups

- **Product showcase** — `Environment` preset + one key light + `MeshReflectorMaterial` floor
- **Outdoor scene** — HDRI environment map + directional sun light with shadows
- **Dark/moody** — minimal ambient + colored point lights + bloom post-processing
- **Stylized** — `MeshToonMaterial` + flat directional light, no environment map
