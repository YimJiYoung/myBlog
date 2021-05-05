---
title: npm은 어떻게 동작하는가
date: 2021-05-05 20:05:54
category: npm
thumbnail: { thumbnailSrc }
draft: false
---

> 이 글은 [How npm Works](https://npm.github.io/how-npm-works-docs/index.html)를 번역하고 정리한 글입니다.  
> npm2와 npm3의 동작에 대해서 설명합니다.

## 패키지와 모듈

**패키지**는 **`package.json`에 의해 기술되는 파일이나 디렉토리**를 의미합니다. npm 레지스트리에 등록되기 위해서는 `package.json` 파일을 반드시 포함해야 합니다. **모듈**은 **Node.js의 `require()` 함수에 의해 로드될 수 있는** (`node_modules` 디렉토리 내에 존재하는) **파일이나 디렉토리**를 의미합니다.

### 모든 모듈은 패키지가 아니다

`node_modules` 내에 `foo.js` 파일을 생성하고 `var f = require('foo.js')`라는 프로그램을 작성하면 모듈(`foo.js`)이 로드됩니다. `foo.js`는 모듈이지만 `package.json`을 포함하지 않으므로 package가 아닙니다.

### 모든 패키지는 모듈이 아니다

`index.js`를 갖지 않거나 `package.json`에 `"main"` 필드가 없는 패키지는 모듈이 아닙니다. `node_modules`에 설치되더라도 `require()`로 로드할 수 없습니다.

### 의존성 지옥

A, B, C라는 모듈이 있다고 가정해봅시다. A는 B v1.0, C는 B v2.0에 의존합니다.

![https://npm.github.io/how-npm-works-docs/gitbook/images/deps1.png](https://npm.github.io/how-npm-works-docs/gitbook/images/deps1.png)

이제  A와 C 모듈를 필요로 하는 어플리케이션을 생성해봅시다.

![https://npm.github.io/how-npm-works-docs/gitbook/images/deps2.png](https://npm.github.io/how-npm-works-docs/gitbook/images/deps2.png)

이때 패키지 매니저는 모듈 B의 어떤 버전을 제공해야할까요?

![https://npm.github.io/how-npm-works-docs/gitbook/images/deps3.png](https://npm.github.io/how-npm-works-docs/gitbook/images/deps3.png)

## npm2

npm은 하나의 B 모듈을 포함하는 것보다는 두가지 버전의 B 모듈 모두 트리 내에 포함하는 방식으로 동작합니다. 각 버전의 모듈은 그것에 의존하는 모듈 내에 존재하게 됩니다.

![https://npm.github.io/how-npm-works-docs/gitbook/images/deps4.png](https://npm.github.io/how-npm-works-docs/gitbook/images/deps4.png)

터미널에서는 다음과 같이 보입니다.

![https://npm.github.io/how-npm-works-docs/gitbook/images/tree.png](https://npm.github.io/how-npm-works-docs/gitbook/images/tree.png)

## npm3

npm3는 npm2와 다른 방식으로 동작합니다. npm2가 모든 의존성을 중첩된 방식으로 설치하지만 npm3는 의존성을 플랫하게 설치하여 중첩으로 인한 중복과 트리의 깊이를 줄이기 위해 노력합니다. 즉, 의존성 패키지가 의존하는 다른 패키지를 같은 레벨에 두는 것입니다. 또한 설치 순서에 따라서 `node_modules` 디렉토리 구조가 달라질 수 있습니다.

이전의 예시와 같이 B v1.0 모듈에 의존하는 A 모듈이 있다고 합시다. npm install을 통해 A 모듈을 설치하면 npm3는 두 모듈을 `node_modules` 내 최상위 위치에 같이 둡니다.

![https://npm.github.io/how-npm-works-docs/gitbook/images/npm3deps2.png](https://npm.github.io/how-npm-works-docs/gitbook/images/npm3deps2.png)

추가로 B v2.0에 의존하는 C 모듈을 설치하려고 하면 B v1.0은 이미 최상위 의존성으로 존재하므로 B v2.0을 같은 위치에 설치할 수 없습니다. 이때는 npm2와 같은 방식으로 중첩된 위치(C 모듈 내부)에 설치하게 됩니다.

![https://npm.github.io/how-npm-works-docs/gitbook/images/npm3deps4.png](https://npm.github.io/how-npm-works-docs/gitbook/images/npm3deps4.png)

이를 터미널을 통해 확인하면 다음과 같습니다.

![https://npm.github.io/how-npm-works-docs/gitbook/images/npm3tree.png](https://npm.github.io/how-npm-works-docs/gitbook/images/npm3tree.png)

추가로 C 모듈과 같이 B v2.0에 의존하는 D 모듈을 새로 설치하려한다면 이전과 같이 동작합니다. B v1.0이 이미 최상위 의존성으로 존재하기 때문에 B v2.0을 최상위 의존성으로 설치할 수 없고 이미 모듈 C에 같은 복제본이 존재하지만 D 모듈 내의 중첩된 의존성으로 설치하게 됩니다.

![https://npm.github.io/how-npm-works-docs/gitbook/images/npm3deps6.png](https://npm.github.io/how-npm-works-docs/gitbook/images/npm3deps6.png)

하지만 이미 최상위 의존성으로 존재하는 B v1.0에 의존하는 E 모듈을 설치하려 한다면 중첩된 형태로 설치하는 것이 아니라 최상위 의존성으로 존재하는 B 모듈을 공유합니다.

![https://npm.github.io/how-npm-works-docs/gitbook/images/npm3deps8.png](https://npm.github.io/how-npm-works-docs/gitbook/images/npm3deps8.png)

이제 A 모듈을 B v2.0에 의존하는 v2.0으로 업그레이드하려 합니다. 어떤 일이 벌어질까요? 여기서 중요한 것은 **설치하는 순서가 중요하다는 것**입니다.

![https://npm.github.io/how-npm-works-docs/gitbook/images/npm3deps9.png](https://npm.github.io/how-npm-works-docs/gitbook/images/npm3deps9.png)

A v1.0가 `package.json`을 통해 가장 처음으로 설치되었다하더라도, A v2.0은 가장 최신에 설치되는 패키지가 될 것입니다.

결과적으로 npm3는 `npm install mod-a@2 --save`를 실행하면  다음과 같이 동작합니다.

- A v1.0을 삭제한다.
- A v2.0을 설치한다.
- E v1.0이 B v1.0에 의존하고 있으므로 그대로 둔다.
- B v1.0은 이미 최상위 의존성 모듈로서 존재하기 때문에 v2.0을 중첩된 의존성으로 A v2.0 아래에 둔다.

![https://npm.github.io/how-npm-works-docs/gitbook/images/npm3deps10.png](https://npm.github.io/how-npm-works-docs/gitbook/images/npm3deps10.png)

마지막으로 A 모듈과 같이 E 모듈을 B v2.0에 의존하는 v2.0으로 업그레이드하려 합니다.

![https://npm.github.io/how-npm-works-docs/gitbook/images/npm3deps11.png](https://npm.github.io/how-npm-works-docs/gitbook/images/npm3deps11.png)

npm3는 다음과 같이 동작합니다.

- E v1.0을 삭제한다.
- E v2.0을 설치한다.
- B v1.0에 의존하는 모듈이 더이상 없으므로 B v1.0을 삭제한다.
- B 모듈이 최상위 위치의 디렉토리에 존재하지 않기 때문에 B v2.0을 해당 위치에 설치한다.

![https://npm.github.io/how-npm-works-docs/gitbook/images/npm3deps12.png](https://npm.github.io/how-npm-works-docs/gitbook/images/npm3deps12.png)

이제 B v2.0은 모든 디렉토리에 존재하게 됩니다. 이런 중복이 발생하는 상황은 바람직하지 않습니다. 이때 다음의 커맨드를 실행할 수 있습니다.

```bash
npm dedupe

```

이 커맨드는 B v2.0에 의존하는 모든 패키지에 대해 최상위 위치에 존재하는 B v2.0를 사용하고 중첨된 복사본을 삭제하도록 합니다.

![https://npm.github.io/how-npm-works-docs/gitbook/images/npm3deps13.png](https://npm.github.io/how-npm-works-docs/gitbook/images/npm3deps13.png)

## 마치며

최근에 패키지의 버전 관련 문제로 골머리를 앓게 되면서 패키지 매니저가 어떻게 동작하는지에 대해 알아보던 차에 좋은 글을 찾아서 이 번역글을 작성하게 되었습니다. 현재는 npm이 7 버전까지 나왔기 때문에 이 글에 작성된 방식보다 더 효율적인 방식으로 동작하고 있겠지만 기본적인 동작 원리는 변하지 않았으리라고 생각합니다. 이 부분에 대해서는 추후 공부를 더 하고 글을 추가로 작성할 수 있으면 좋겠습니다. 😊