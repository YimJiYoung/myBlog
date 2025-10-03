---
title: 번들 사이즈 최적화
date: 2025-01-28 17:10:33
category: web
thumbnail: { thumbnailSrc }
draft: false
---

## Code Splitting

코드 분할은 코드를 필요한 시점에 필요한 코드만 불러오는 기술입니다. 초기에는 최소한의 코드만 로드하고, 사용자가 특정 페이지나 기능에 접근할 때 필요한 코드만 추가로 로드하는 방식입니다. dynamic import를 사용하여 별도의 번들로 분할할 수 있습니다. (webpack, rollup와 같은 대부분의 번들러에서 지원됩니다.) 

```jsx
form.addEventListener("submit", e => {
  e.preventDefault();
  import('moduleA')
    .then(module => module.default)
    .then(() => someFunction())
    .catch(handleError());
});

const someFunction = () => {
    // moduleA 사용
}
```

위의 예제에서 모듈(`moduleA`)을 구성하는 코드는 초기 번들에 포함되지 않으며 지연 로드되거나 양식 제출 후 필요할 때만 사용자에게 제공됩니다.

## Tree shaking

트리쉐이킹은 사용되지 않는 코드(dead-code)를 제거하여 최종 번들 크기를 줄이는 기술입니다. 이 기술은 정적인 ES module(ESM) 문법에 의존합니다.

```jsx
// utils의 모든 유틸을 불러옵니다.
import utils from "utils";

// utils의 일부 유틸만 불러옵니다.
// 명시적으로 가져오지 않은 모듈을 삭제하는 트리쉐이킹이 가능해집니다.
import { unique, implode, explode } from "utils";
```

### ESM 모듈 사용하기

트리쉐이킹은 정적인 ESM 문법에 의존하며 CommonJS(CJS)는 트리쉐이킹이 어렵습니다.

CJS에서는 `require`로 전체 모듈을 불러오고, 모듈을 동적으로 사용할 수 있습니다. 

```jsx
// 전체 utils 객체를 불러옵니다.
const utils = require('./utils');

// 동적으로 utils를 불러올 수 있습니다.
const util = require(`./utils/${utilName}`);
```

이처럼 CJS는 모듈을 동적으로 다룰 수 있어, 빌드 타임에 모듈 간의 관계를 분석하기 어렵습니다. 모듈 간의 관계을 런타임에만 파악할 수 있기 때문에 트리쉐이킹을 적용하기가 어렵습니다.

반면, **ESM**은 정적인 구조로 작성되어 있어 빌드 타임에 모듈 간의 관계를 분석할 수 있습니다. 덕분에 사용되지 않는 코드를 쉽게 제거할 수 있습니다.

그렇다면 사용하는 패키지가 CJS인지 ESM인지 어떻게 알까요?

**패키지가 CJS인지 ESM인지 확인하는 방법**

`.js` 파일의 모듈 포맷은 package.json의 `type` field에 따라 결정됩니다.

- `"commonjs"` : `.js` 는 CJS로 해석됩니다. (기본값)
- `"module"` : `.js` 는 ESM으로 해석됩니다.

**패키지가 CJS와 ESM을 모두 제공하는 경우**

일부 패키지는 **conditional exports**를 사용하여 CJS와 ESM을 모두 제공합니다.

이 방법을 통해 런타임 환경에 따라 적절한 모듈 형식을 선택할 수 있습니다.

자세한 내용은 [Node.js의 Conditional Exports 문서](https://nodejs.org/api/packages.html#conditional-exports)를 참고하세요.

```jsx
// package.json
{
  "exports": {
    // import or import()로 불러온 모듈
    "import": "./index-module.js",
    // require()로 불러온 모듈
    "require": "./index-require.cjs"
  }
} 
```

### 사이드 이펙트 없애기

트리쉐이킹이 효과적으로 동작하려면 사용하는 패키지나 코드가 사이드 이펙트(side effects)를 갖고 있지 않아야 합니다. 사이드 이펙트란 코드가 외부 환경에 영향을 주거나 외부에서 영향을 받아서, 실행 결과를 예측하기 어려운 경우를 말합니다. 예를 들어, 브라우저 호환성을 위해 전역 객체를 수정하는 폴리필 코드나, 전역 스타일을 적용하는 CSS 파일이 이에 해당합니다.

Webpack은 사이드 이펙트가 없는 코드 ****를 안전하게 제거합니다. package.json 파일에 `sideEffects` 속성을 추가하여 Webpack에 힌트를 줄 수 있습니다. 

1. 사이드 이펙트가 전혀 없는 경우
    
    ```jsx
    {
      "sideEffects": false
    }
    ```
    
    Webpack은 사용되지 않는 모든 코드를 제거합니다.
    
2. 일부 파일만 사이드 이펙트를 가지는 경우
    
    ```jsx
    {
      "sideEffects": ["./src/some-side-effectful-file.js"]
    }
    ```
    
    이 설정은 특정 파일만 사이드 이펙트를 가지며, 나머지 코드는 안전하게 제거할 수 있음을 Webpack에 알려줍니다.
    

**참고**

- [https://web.dev/articles/reduce-javascript-payloads-with-code-splitting](https://web.dev/articles/reduce-javascript-payloads-with-code-splitting?hl=ko)
- [https://web.dev/articles/reduce-javascript-payloads-with-tree-shaking](https://web.dev/articles/reduce-javascript-payloads-with-tree-shaking)
- [https://toss.tech/article/commonjs-esm-exports-field](https://toss.tech/article/commonjs-esm-exports-field)
- [https://rollupjs.org/introduction/#tree-shaking](https://rollupjs.org/introduction/#tree-shaking)
- [https://www.heropy.dev/p/pd8CoS](https://www.heropy.dev/p/pd8CoS)