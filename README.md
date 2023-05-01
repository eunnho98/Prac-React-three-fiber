# Prac-React-three-fiber

# Canvas

React Three Fiber을 정의하기 위한 Scene 생성

## Properties

- gl: 렌더러에 옵션 지정, R3F은 Canvas Component를 쓰면 자동으로 default 렌더러가 할당되는데 만약 나만의 렌더러를 지정하고 싶다면 활용
    - WebGLRenderer을 쓰고 antialias를 true로 한 후 여러 설정을 하고 싶을 때
- camera: 카메라에 옵션 지정, R3F의 카메라는 default로 { fov: 75, near: 0.1, far: 1000, position: [0, 0, 5] }인데 나만의 카메라를 쓰고싶다면 지정가능
- scene: Scene에 옵션 지정, 역시나 default Scene이 있지만 나만의 Scene을 쓰고 싶을 때 사용
    - scene과 renderer에 모두 color 설정이 되어있다면 scene의 색이 우선 지정된다.

```jsx
<Canvas
      camera={{ position: [0, 0, 2], aspect: 2 } }}
      scene={{ background: 'black' }}
      gl={(canvas) => {
        const renderer = new WebGLRenderer({ canvas, antialias: true })
        renderer.setPixelRatio(window.devicePixelRatio > 1 ? 2 : 1)
        return renderer
      }}
>
...
```

카메라의 포지션을 변경하고 scene의 색을 변경하고 내가 설정한 나만의 WebGLRenderer을 쓸 때

즉 three.js의 camera, scene, renderer가 가지고있는 프로퍼티에 대해 잘 알아야 결국 사용 가능

- shadows, true라고 하면 renderer.shadowMap.enable = true라고 하는 것, 즉 default Shadow인 PCFsoft에 넘길 수 있는 property.
    - 만약 다른 shadowMap Type을 쓰려면 gl에 설정해줘야한다.(renderer.shadowType.~~)
- raycaster: R3F의 default raycaster의 origin은 Vector3를 쓴다고 함.

```jsx
// raycaster에 속성을 지정하는 코드
// 기본은 Vector3인데 이벤트가 발생하면 Vector2(마우스클릭)이 되도록
import { Canvas } from 'react-three-fiber';
import * as THREE from 'three';

function MyCanvas() {
  const raycaster = new THREE.Raycaster(
    new THREE.Vector3(0, 0, 0), // origin
    new THREE.Vector3(0, 0, -1) // direction
  );

  function handleMouseMove(event) {
    const mouse = new THREE.Vector2();
    mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
    mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;

    raycaster.setFromCamera(mouse, camera);
    // perform intersection tests here using raycaster
  }

  return (
    <Canvas raycaster={raycaster}>
      {/* ... */}
    </Canvas>
  );
}
```

- frameloop: 렌더링하는 한 프레임마다 발생하는 콜백함수 지정 가능, 애니메이션이나 동적 프로퍼티를 쓸 때 사용한다고 함
    - 어떤 함수로 scene을 업데이트하는데 멈추었다 다시 trigger하려면 invalidateFrameloop() 함수를 써야한다.

```jsx
function MyCanvas() {
  const [paused, setPaused] = useState(false);

  function handleFrame() {
    // Update scene here
  }

  function handlePause() {
    setPaused(true);
  }

  function handleResume() {
    setPaused(false);
    // Call invalidateFrameloop() to resume the animation
    invalidateFrameloop();
  }

  return (
    <Canvas frameloop={!paused && handleFrame}>
      <mesh>
        <boxBufferGeometry />
        <meshStandardMaterial />
      </mesh>
      <button onClick={handlePause}>Pause</button>
      <button onClick={handleResume}>Resume</button>
    </Canvas>
  );
}
```

- orthographic: true라고 하면 PerspectiveCamear가 아닌 OrthographicCameara 사용
- **onCreated:** canvas가 불러와질 때 호출되는 콜백함수 지정 가능
- onPoinerMissed: cavas내 Interactive한 오브젝트를 클릭하지 않았을 때 발생하는 콜백함수 지정

## render defaults

Canvas를 쓰면 다음과 같은 default가 따라 나온다고 한다.

- WebGLRenderer that antialias = true, alpha = true, powerPreference=’high-performance’
- Perspective Camera(orthographic이 true라면 Orthographic Camera)
- shadows가 true라면 PEFSoftShadowMap
- THREE.Scene, THREE.Raycaster가 default로 따라나온다 → 속성은 내가 직접 지정해야함

# Objects, properties and contructor arguments
## Object 3D

Object 3D 객체에는 다음과 같은 것들이 있음

→ Mesh, Group, Camera, Light, Sprite, Line, Points

그리고 공식문서를 참고하면 여러 Property와 Method가 있다

[three.js docs](https://threejs.org/docs/#api/en/core/Object3D.position)

이 프로퍼티와 메서드를 R3F를 쓸 때 프로퍼티로 넘길 수 있음

```jsx
<mesh
  visible
  userData={{ hello: 'world' }}
  position={new THREE.Vector3(1, 2, 3)}
  rotation={new THREE.Euler(Math.PI / 2, 0, 0)}
  geometry={new THREE.SphereGeometry(1, 16, 16)}
  material={new THREE.MeshBasicMaterial({ color: new THREE.Color('hotpink'), transparent: true })}
/>
```

위처럼 mesh의 경우 각종 프로퍼티와 메서드가 있지만, 위와 같이 사용하면 재사용성이 떨어지고 mesh는 geometry, material로 구성되므로 다음과같이 바꿀 수 있다.

```jsx
<mesh visible userData={{ hello: 'world' }} position={[1, 2, 3]} rotation={[Math.PI / 2, 0, 0]}>
  <sphereGeometry args={[1, 16, 16]} />
  <meshStandardMaterial color="hotpink" transparent />
</mesh>
```

그리고 nested attributes에 접근할 땐(mesh.rotation.x) dash로 접근할 수 있다.

```jsx
<mesh rotation-x={1} material-uniforms-resolution-value={[512, 512]} />
```

## 생성자 arguments

Three.js에서는 const geometry = new THREE.SphereGeometry(1, 32)처럼 생성자에 인자를 전달했지만 R3F에서는 **args**로 전달한다.

```jsx
<sphereGeometry args={[1, 32]} />
```

## Attach

Three.js Object를 리액트 구성요소에 연결하는데 사용

```jsx
<mesh>
  <meshBasicMaterial attach="material">
  <boxGeometry attach="geometry">
```

원래 위처럼 일일이 다 써줘야 하는데 THREE.Material과 THREE.Geometry는 attach 생략 가능, 어차피 Mesh를 정의할 때 꼭 필요한 것이므로

### 여러 예시

```jsx
// Attach bar to foo.a
<foo>
  <bar attach="a" />

// Attach bar to foo.a.b and foo.a.b.c (nested object attach)
<foo>
  <bar attach="a-b" />
  <bar attach="a-b-c" />

// Attach bar to foo.a[0] and foo.a[1] (array attach is just object attach)
<foo>
  <bar attach="a-0" />
  <bar attach="a-1" />

// Attach bar to foo via explicit add/remove functions
<foo>
  <bar attach={(parent, self) => {
    parent.add(self)
    return () => parent.remove(self)
  }} />

// The same as a one liner
<foo>
  <bar attach={(parent, self) => (parent.add(self), () => parent.remove(self))} />
```

attach도 콜백함수를 받을 수 있음, 인자로(부모, 나자신)

return은 내가 부모와 바인드가 끝날 때 실행

실사용 예시

```jsx
<mesh>
  {colors.map((color, index) => <meshBasicMaterial key={index} attach={`material-${index}`} color={color} />}
</mesh>
```

## primitve components

메쉬같은 Three.js 객체를 따로 React 구성 요소를 만드는 것보다 가볍게 렌더링 가능

```jsx
const mesh = new THREE.Mesh(geometry, material)

function Component() {
  return <primitive object={mesh} position={[10, 0, 0]} />
```

- object에 렌더링할 객체를 넣고 위치 등의 여러 props를 전달 가능
- attach를 사용하여 이 객체를 다른 객체의 자식으로도 넣을 수 있다.

# Hooks
Hook들은 Canvas element 내애세만 사용할 수 있음!

```jsx
// 불가능
import { useThree } from '@react-three/fiber'

function App() {
  const { size } = useThree() // This will just crash
  return (
    <Canvas>
      <mesh>

// 가능
function Foo() {
  const { size } = useThree()
  ...
}

function App() {
  return (
    <Canvas>
      <Foo />
```

## useThree

특정 Canvas의 카메라, Scene, Renderer등의 정보를 나타냄

```jsx
export default function Foo() {
  const set = useThree((state) => state.set)
  const state = useThree()
  const get = useThree((state) => state.get)
  console.log(get())
  console.log(state)
  return
}
```

- console.log(get())과 console.log(state)는 같은 의미이지만 state는 어떤 순간의 snapshot을 담고있는 반면 get()은 state를 계속 체크할 수 있다.
- set을 통해 Canvas의 특정 속성을 변경할 수도 있다) 카메라의 종류 변경 등
- 여러 Canvas가 있을 때 특정 Canvas에 useThree를 쓰는 함수를 nesting 하면 그 Canvas에 대한 정보만 나타낸다.

## useFrame

렌더링된 모든 프레임에서 effect를 실행하거나 컨트롤을 업데이트 하는 등의 코드 실행 가능

useFrame 내에서 업데이트할 땐 ref로 해야함, state로 하면 안됨

→ state로 하면 재렌더링이 이루어지고, 렌더링을 할 땐 Virtual DOM과 DOM을 비교하는 과정이 포함된다

→ ref로 하면 직접 DOM을 조작하므로 렌더링이 발생하지 않아 성능상 이점이 있다.

```jsx
useFrame((state, delta) => {
    ref.current.rotation.x += 1 * delta
    ref.current.rotation.y += 0.5 * delta
  })
```

state에 현재 scene에 대한 정보가 담겨있고, delta는 매 프레임마다의 간격이 담긴다.

참고로 ref.current.clock에 여러 time이 있으니 필요할 때마다 쓰면 된다.
