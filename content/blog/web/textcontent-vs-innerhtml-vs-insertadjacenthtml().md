---
title: textContent vs innerHTML vs insertAdjacentHTML()
date: 2021-03-06 11:03:58
category: web
thumbnail: { thumbnailSrc }
draft: false
---

어떤 요소의 text나 자식 요소에 대한 HTML 컨텐츠를 가져오고자 할 때 혹은 삽입하고자 할 때 사용할 수 있는 DOM API로는 [Node.textContent](https://developer.mozilla.org/en-US/docs/Web/API/Node/textContent), [Element.innerHTML](https://www.notion.so/Element-innerHTML-91abfdaec19a4cde88301a705bb48ca7), [Element.insertAdjacentHTML()](https://developer.mozilla.org/en-US/docs/Web/API/Element/insertAdjacentHTML) 이 있다. 각 API에 대해 어떻게 동작하는지, 어떤 때에 사용하는 것이 좋은지 각각의 특징을 알아보도록 하자.

## textContent

textContent는 해당 요소와 모든 자식 요소의 text를 가져와서 반환한다. `<script>` 와 `<style>`, `display: none`으로 스타일이 지정된 요소의 text까지 가져오는 것에 주의해야 한다.  textContent로 설정 시 해당 요소의 모든 자식 노드를 삭제하고 하나의 텍스트 노드로 대치한다.

### Example

```html
<p id="source">
  <style>#source { color: red; }</style>
아래에서<br>이 글을<br>어떻게 인식하는지 살펴보세요.
  <span style="display:none">숨겨진 글</span>
</p>
```

**textContent 결과**

```
  #source { color: red; }
아래에서이 글을어떻게 인식하는지 살펴보세요.
숨겨진 글

```

**innerText 결과**

innerText는 textContent와 달리 화면에 보이는 text를 반환한다. `<br>`과 숨겨진 요소까지 인식하고 있는 것을 확인할 수 있다.

```
아래에서
이 글을
어떻게 인식하는지 살펴보세요.
```

## innerHTML

해당 요소 내의 HTML 문자열을 가져오거나 설정할 수 있다. innerHTML의 값을 설정하면 모든 자식 노드가 제거되고 입력된 HTML 문자열을 파싱하고 생성된 노드로 대체된다. 이때 입력된 전체 HTML 문자열에 대해 파싱하고 DOM tree 구성하고 삽입하는 추가 작업이 발생하기 때문에 전체 변경이 아니라 부분적인 변경이 필요한 경우나 삽입을 하는 경우는 `Node.appendChild()`나 `insertAdjacentHTML()` 을 사용하는 것이 더 효율적이다.

### 보안 문제(XSS)

innerHTML에 `<script>` 를 포함하면 해당 스크립트가 실행되기 때문에 cross-site-scripting의 위험이 있다. HTML5에서는 innerHTML로 삽입된 `<script>` 실행하지 않도록 되어 있지만 다른 방법(`<img src='x' onerror='alert(1)'>`)으로 스크립트를 실행되도록 할 수 있으므로 **사용자 입력 문자열을 innerHTML에 추가해서는 안된다**. 단순 문자열 삽입의 경우는 textContent를 사용하는 것이 더 권장된다.

## insertAdjacentHTML()

요소의 지정된 위치에 HTML 문자열을 파싱하여 생성된 노드를 삽입한다. innerHTML과 달리 요소 내에 있는 기존의 자식 요소를 건드리지 않기 때문에 더 빠르게 동작한다. innerHTML과 같이 XSS의 위험이 존재하므로 사용자 입력 문자열을 삽입하지 않도록 주의하자.

```jsx
element.insertAdjacentHTML(position, text);

<!-- beforebegin -->
<p>
  <!-- afterbegin -->
  foo
  <!-- beforeend -->
</p>
<!-- afterend -->
```

## 참고

- [https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/innerText](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/innerText)
- [https://developer.mozilla.org/en-US/docs/Web/API/Node/textContent](https://developer.mozilla.org/en-US/docs/Web/API/Node/textContent)
- [https://developer.mozilla.org/en-US/docs/Web/API/Element/innerHTML](https://developer.mozilla.org/en-US/docs/Web/API/Element/innerHTML)
- [https://developer.mozilla.org/en-US/docs/Web/API/Element/insertAdjacentHTML](https://developer.mozilla.org/en-US/docs/Web/API/Element/insertAdjacentHTML)