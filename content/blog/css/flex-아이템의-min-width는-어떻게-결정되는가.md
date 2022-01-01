---
title:  🐞 플렉스 아이템의 크기가 컨테이너보다 커지는 오류
date: 2022-01-01 15:01:89
category: css
thumbnail: { thumbnailSrc }
draft: false
---

## 들어가며
실무를 하면서 CSS 플렉스 속성을 자주 사용하게 되는데 최근에 예상치 못한 플렉스 아이템의 크기가 플렉스 컨테이너보다 커져서 오버플로우되는 오류🐞를 목격하게 되었고 그 원인에 대해서 알아보면서 찾은 해결 방법을 공유하려고 합니다. 다음은 오류가 발생하는 상황을 재현한 코드입니다.
  
## Case 1
<iframe height="300" style="width: 100%;" scrolling="no" title="Untitled" src="https://codepen.io/yimjiyoung/embed/mdMQKZM?default-tab=html%2Cresult" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href="https://codepen.io/yimjiyoung/pen/mdMQKZM">
  flex-overflow-1</a> by Mia (<a href="https://codepen.io/yimjiyoung">@yimjiyoung</a>)
  on <a href="https://codepen.io">CodePen</a>.
</iframe>

`display: flex` 속성을 갖는 너비 `300px`의 메인 컨테이너에서 두 개의 아이템을 가지고 있고, 두번째 아이템의 너비는 `100px`로 고정되며 첫번째 아이템은 `flex: 1`을 주어 나머지 영역(`200px`)을 차지합니다. 하지만 첫번째 아이템 내의 10개의 항목을 가진 목록이 포함되면서 너비가 `200px`보다 커지면서 오버플로우가 발생합니다. 목록 스타일에 `overflow: auto` 속성을 주었기 때문에 `200px` 내에서 스크롤되는 동작을 예상했는데 그렇지 않고 컨테이너 영역보다 큰 크기를 차지하게 됩니다. 왜 이런 상황이 발생한 것일까요? 원인은 플렉스 아이템이 줄어들 수 있는 최소 크기(`min-width`)가 컨텐츠의 최소 크기(`min-content`)로 설정되기 때문입니다. 

### 플렉스의 아이템의 최소 크기는 어떻게 결정되는가

`min-width/height`의 기본값은 `auto`이며 플렉스 아이템이 아닌 경우 `auto`의 값은 `0`과 동일합니다. 즉, 플렉스 아이템이 아닌 요소(+ 스크롤 가능한 플렉스 아이템)는 크기가 `0`까지 줄어들 수 있습니다. 반면 플렉스 아이템의 경우는 컨텐츠의 `min-content`입니다. `min-content`는 컨텐츠가 오버플로우없이 차지할 수 있는 최소 크기를 의미합니다. 이때문에 이전 코드에서는 플렉스 아이템이 해당 요소의 컨텐츠보다 작아질 수 없었습니다.

### 해결 방법

원인을 알았기 때문에 이제 해결 방법은 간단합니다. 플렉스 아이템이 컨텐츠 크기보다 더 작아질 수 있게 설정하면 됩니다. 위의 코드로 돌아가서 `min-width: 0` 속성을 플렉스 아이템에 추가해봅시다.

## Case 2

<iframe height="300" style="width: 100%;" scrolling="no" title="Untitled" src="https://codepen.io/yimjiyoung/embed/RwZqJzg?default-tab=html%2Cresult" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href="https://codepen.io/yimjiyoung/pen/RwZqJzg">
  flex-overflow-2</a> by Mia (<a href="https://codepen.io/yimjiyoung">@yimjiyoung</a>)
  on <a href="https://codepen.io">CodePen</a>.
</iframe>

이번에는 텍스트로 인해 오버플로우가 발생하는 오류가 발생하는 상황입니다. 영문의 경우 기본적으로 띄어쓰기나 `-`와 같은 문장 부호에서 줄바꿈이 일어날 수 있는데 위의 코드와 같이 띄어쓰기 없이 이어진 긴 단어로 인해 오버플로우가 발생하게 됐습니다.

### 해결 방법
 이 경우에는 간단하게 `word-break: break-all` 속성을 추가하여 어디서든 줄바꿈이 일어날 수 있도록 설정하여 해결할 수도 있지만 `overflow-wrap` 속성을 사용할 수도 있습니다. `overflow-wrap` 속성으로 텍스트가 오버플로우되는 것을 막기 위해서 임의의 지점에서 줄바꿈을 삽입할 수 있도록 설정할 수 있습니다. 사용할 수 있는 속성 값으로는 기본값인 `normal`, `break-word`와 `anywhere`가 있습니다. `break-word`와 `anywhere`를 사용하여 오버플로우를 방지하기 위해 임의의 지점에 줄바꿈이 일어나도록 설정합니다. 그러나 `overflow-wrap: break-word` 속성을 추가해도 오류가 해결되지 않고 `anywhere`를 사용했을 때 해결되는 것을 확인할 수 있습니다. 이유는 `anywhere` 사용 시엔 임의의 지점에서 줄바꿈이 일어날 수 있다는 점이 `min-content` 계산 시 고려되기 때문입니다. `break-word`의 경우엔 고려되지 않기 때문에 여전히 긴 텍스트가 `min-content` 계산 시에 포함되기 때문에 그 너비보다 작아질 수 없습니다. 따라서 Case1의 해결방법처럼 `min-width: 0`을 추가해줘야 합니다. `anywhere`를 사용하는 것이 가장 바람직해보이지만 안타깝게도 [브라우저 지원 비율이 낮습니다.](https://caniuse.com/?search=overflow-wrap%3Aanywhere) 

## 마치며
두 가지 상황을 보면서 플렉스와 `min-width`, `overflow-wrap` 속성, `min-content`에 대해서 다루어 보았습니다. 오류 상황에 대해 원인을 파악해보면서 CSS 명세 문서까지 보기도 하고 그동안 애용하고 있던 플렉스에 대해서 더 잘 알 수 있게 되었던 기회였습니다. 읽으시는 분들께도 도움이 되는 내용이었으면 좋겠습니다 ☺️

## 참고 
https://www.w3.org/TR/css-flexbox-1/#min-size-auto
https://drafts.csswg.org/css-text/#overflow-wrap-property