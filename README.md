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
