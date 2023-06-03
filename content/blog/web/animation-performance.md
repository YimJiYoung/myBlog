---
title: Animation Performance
date: 2023-06-03 17:06:93
category: web
thumbnail: { thumbnailSrc }
draft: false
---


# 애니메이션 성능

웹 애니메이션 성능에 기여하는 두 가지 주요한 요인이 있는데, **렌더링**과 **하드웨어 가속**입니다. 

렌더링은 DOM을 업데이트하고 화면에 변화를 반영하기까지 걸리는 처리 과정을 의미합니다. 이는 모든 애니메이션의 성능에 영향을 미칩니다.

하드웨어 가속은 CPU가 아닌 GPU에서 애니메이션을 실행하도록 위임하는 것입니다. 애니메이션이 메인 스레드에 의해 영향을 받지 않기 때문에 React 렌더링과 같은 다른 JS 작업과 동시에 실행되더라도 덜 버벅이게 됩니다.

## Rendering

요소의 스타일을 업데이트할 때 브라우저는 변경 사항을 반영하기 위해 페이지를 리렌더링해야 합니다. 이때 다음의 과정을 거칠 수 있습니다.

1. **Recalculating style** — 요소에 어떤 CSS 선언이 적용되어야 하는지 알아냅니다.
2. **Layout** — 페이지에 각 요소가 어디에 위치해야 하는지 계산합니다. (각 요소의 위치, 크기 계산)
3. **Paint** — 이제 요소를 화면에 그립니다. 모든 픽셀에 대해 어떤 색상을 가지는지 처리하는 과정(”레스터화”)을 거칩니다. 이때 레이어로 분리된 웹 페이지의 각 부분에 대해 레스터화합니다.
4. **Compositing** — 마지막으로, 이전에 그린 레이어를 웹 페이지로 합성합니다.

  네번째 단계인 합성(Compositing)에서 브라우저는 이전의 프레임을 재활용합니다. 이는 스크롤 성능을 향상시키기 위해 개발됐습니다. 초기의 웹은 뷰포트 안쪽만을 레스터화하고 사용자가 웹 페이지를 스크롤하면 이미 레스터화한 프레임을 움직이고 나머지 빈 부분에 대해 추가로 레스터화했는데, 이는 매우 느렸습니다. 따라서 스크롤 시에 레스터화 단계를 스킵하고 미리 레스터화된 레이어에 대해 레이어를 움직이고 합성하는 방식으로 개선됐습니다. 합성은 이미 계산된 프레임에 대해 위치를 옮기거나 회전시키거나 하는 등의 일을 하기 때문에 많은 계산이 필요하지 않으며, 메인 스레드와 별개로 작동할 수 있습니다.

### Slow rendering

예를 들어, 요소의 `height` 을 업데이트한다면 해당 요소가 자매나 자식 요소의 크기 또는 위치에 영향을 미칠 수 있기 때문에 영향을 미치는 모든 요소에 대해 레이아웃을 다시 계산해야 합니다. 레이아웃이 변할 때 페인트와 합성도 다시 해야합니다. `height`, `border-width`, `padding`, `position` 와 같은 많은 속성이 레이아웃에 영향을 미칩니다.

부드러운 애니메이션은 1초에 60프레임을 필요로 합니다. 만약 `height`에 애니메이션을 적용한다면 요소의 높이를 16.7ms마다 리렌더할 필요가 있는데, 레이아웃을 동반한 리렌더는 100ms 이상이 될 수도 있습니다 .. 

하지만 만약 요소의 레이아웃이 분리되어 있거나, `position: absolute` 이라면 영향을 미칠 자매 요소가 없기 때문에 부드러운 애니메이션이 가능할 수 있습니다. 가장 좋은 것은 성능이 낮은 기기에서 애니메이션을 테스트해보는 것입니다.

### Fast rendering

가장 좋은 것은 네번째 렌더링 단계인 합성만 트리거하는 스타일에 애니메이션을 적용하는 것입니다. 대부분의 브라우저에서 `transform` 과 `opacity` 가 여기에 속합니다. 브라우저는 이 리스트에 더 많은 스타일을 추가하고 있습니다. 예를 들어, 크롬과 파이어폭스는 `filter` 를 컴포지터에서 온전히 처리하고 있고, [크롬은  `background-color` 나 `clip-path`도 곧 지원할 예정이라고 합니다.](https://developer.chrome.com/blog/hardware-accelerated-animations/)

### Reduce layer size

브라우저가 레이어를 다시 그리는데 걸리는 시간은 레이어의 크기에 비례한다. 그래서 레이어를 작게 만들면 애니메이션에 성능을 향상시킬 수 있다. `will-change` 속성을 통해 브라우저에게 새로운 레이어를 만들도록 힌트를 줄 수 있다.

```jsx
element.style.willChange = "transform"
```

새로운 레이어를 만드는 것에 비용이 없는 것은 아니다. 각 레이어는 GPU의 메모리를 차지하기 때문에 필요할 때에 사용하는 것이 좋다.

## Hardware acceleration

렌더링 성능은 우리가 모든 애니메이션을 할 때 고려해야할 주요한 부분이다. 하지만 애니메이션 과정 자체에는 추가적인 요인이 있으며 그것은 하드웨어 가속화가 될 수 있을 것이다. 

애니메이션은 기본적으로 어떤 지속 시간 동안 두 값을 혼합하는 것이다. 예를 들어, “100px”과 “200px” 사이를 1초 동안 애니메이션을 실행한다면, 0.5 초가 지나면 애니메이션은 “150px”을 계산할 것이다. 이는 매우 간단한 연산이며 렌더링과 비교하면 큰 오버헤드가 아니다. 

하지만, 브라우저에서 이런 값을 계산하는 몇 가지 방법이 있다.

- 자바스크립트로, `requestAnimationFrame` 사용 (react-spring, framer 등 JS 기반 애니메이션 라이브러리가 여기에 해당)
- [Web Animation API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Animations_API/Using_the_Web_Animations_API)
- CSS

자바스크립트 코드는 항상 메인 JS 스레드에서 실행된다. 이는 당신의 애플리케이션이 다른 JS 코드를 동시에 실행하고 있으면, 당신의 애니메이션 코드는 제때 실행되지 못할 가능성이 있고 버벅이는 애니메이션을 야기할 수 있습니다.

하지만, WAAP와 CSS는 일부 애니메이션을 JS 스레드에서 분리하여 컴포지터에서 직접 실행할 수 있습니다. 컴포지터는 대체로 GPU가 되며, 이것을 하드웨어 가속화라고 부릅니다.

애니메이션이 하드웨어 가속화되면 메인 JS 스레드가 바쁜 경우에도 부드러운 애니메이션이 가능합니다.

### Web Animation API이란?

브라우저에 내장된 로우 레벨 애니메이션 API이며, CSS keyframe의 자바스크립트 버전이라고 생각해도 좋습니다. 내부적으로 WAAP는 CSS keyframe과 트랜지션와 같은 구현체를 사용하기 때문에 성능적으로 CSS와 동일합니다.

#### CSS keyframe  vs WAAP 비교

```css
#alice {
  animation: aliceTumbling infinite 3s linear;
}

@keyframes aliceTumbling {
  0% {
    color: #000;
    transform: rotate(0) translate3D(-50%, -50%, 0);
  }
  30% {
    color: #431236;
  }
  100% {
    color: #000;
    transform: rotate(360deg) translate3D(-50%, -50%, 0);
  }
}
```

```javascript
document.getElementById("alice").animate(
  [
    { transform: "rotate(0) translate3D(-50%, -50%, 0)", color: "#000" },
    { color: "#431236", offset: 0.3 },
    { transform: "rotate(360deg) translate3D(-50%, -50%, 0)", color: "#000" },
  ],
  {
    duration: 3000,
    iterations: Infinity,
  }
);
```

## 참고
- [Motion One Performance](https://motion.dev/guides/performance)
- [CSS for JS](https://css-for-js.dev/)