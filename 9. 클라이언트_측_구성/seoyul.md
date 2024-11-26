# 클라이언트 측 구성의 기본

- 클라이언트 측 구성은 집계 계층이 필요하지 않은 대신 주요한 HTML 문서(셸 또는 앱셸)의 위치를 알 필요가 있다.
- 앱셸은 개별적인 마이크로 프런트엔드를 표현하는 스크립트 같은 가장 중요한 자원의 참조를 갖고 있고, 가능한 자기완비적이 되게 하는것이 이상적이며, 전용 서버나 중앙 집중적인 컨텐츠 제공자에서 제공할 수 있다.
- 클라이언트 측 구성은 일반적으로 확립된 메커니즘과 고정 패턴에 의존한다. 가장 편리한 방법중 하나는 웹 컴포넌트이다.
- 웹 컴포넌트를 사용하는 것은 많은 유연성과 자유를 줘서 어떤 애플리케이션 내에서도 특정한 마이크로 프런트엔드를 도입할 수 있게 한다.
- 아키텍처 관점에서 두가지 방법 중 하나를 선택해야 한다.

1. 웹 컴포넌트마다 하나의 스크립트를 갖는것
2. 마이크로 프런트엔드마다 하나의 스크립트를 갖는것

- 전자는 같은 도메인에 속하는 컴포넌트들의 오케스트레이션을 결극 더 복잡하게 만들어 후자가 훨씬 간단하고 좋은 성능을 제공한다.
- 클라이언트측 구성은 널리 사용되는 기존 ㅇ키텍처에 추가하는 패턴으로 도구처럼 사용하는 웹 어플리케이션에서 주로 볼 수 있다.

## 웹 컴폰너트

- 웹 컴포넌트는 웹에 대한 표준 컴포넌트 모델을 제공하려는 피처 집합에 대한 포괄적인 표현으로 자바스크립트 프레임워크의 부상과 클라이언트측 렌더링의 중요성이 커짐에 따라 표준화 되었다.
- 리액트, 뷰, 앵귤러와 같은 SPA 프레임워크에서 제공하는 html 코드 조각들을 컴포넌트화 하여 관리하는 것과 같은 기술을 칭합니다.
- 리액트는 재사용 가능한 UI 조각들을 컴포넌트화 하여 관리하기 용이하게 해주는데, 바닐라 자바스크립트에서도 웹 컴포넌트 기술을 활용하면 가능하다.
- 웹 컴포넌트의 장점은 어떤 프레임워크를 사용하고 있는지 중요하지 않고, 어디에서나 사용할 수 있다.
- 커스텀 html 태그라고도 볼 수 있는데, 길게 늘어져있는 코드블럭을 하나의 태그로 명명하여 가독성 좋게 작성할 수 있다.

```
class MyWebComponent extends HTMLElement {...};
window.customElements.define('my-web-component', MyWebComponent);
```

```
<my-web-component>
    <h1>Hello world!</h1>
</my-web-component>
```

- 스크립트 태그 내부 또는 외부 js파일에 우리가 만들고자 하는 커스텀 DOM 요소를 class 키워드를 통해 만들고, customElements.define()메서드를 통해 해당 클래스를 태그명 작명과 함께 등록한 뒤 사용하고 싶은 곳에 html 태그처럼 쓸 수 있다.

```
class MyComponent extends HTMLElement{
    constructor() {
      super();
      /*called when the class is
      instantiated
      */
    }
 connectedCallback() {
      this.style.color = 'red';

     /*called when the element is
      connected to the page.
      This can be called multiple
      times during the element's
      lifecycle.
      for example when using drag&drop
      to move elements around
     */
   }
 disconnectedCallback() {
   /*called when the element
     is disconnected from the page
   */
   }
}
```

- 커스텀 DOM요소를 만들기 위해 만든 클래스는 HTMLElement 클래스를 상속받아야 커스텀 요소를 html tag와 같이 다룰 수 있다.
- 웹 컴포넌트 라이프사이클 콜백을 통해 생성되고 소멸되기 까지 일정한 주기 가운데 트리거

### Shadow DOM

- 웹 컴포넌트의 장점이자 특징이 캡슐화인데, shadow DOM을 활용해 이를 가능하게 할 수 있다.
- shadowDOM에서 일어나는 스타일링, 자식의 append, attribute의 설정 등은 외부의 트리에 어떠한 영향도 끼치지 않는다.

```
let shadow = this.attachShadow({mode: 'open'});
```

```
<!DOCTYPE html>
<html>
<body>
<template>
    <h1>Hello Rick!</h1>
</template>
<my-component></my-component>
<script>
 class MyComponent extends HTMLElement {
 constructor() {
      super();
      this.addEventListener('click',
      () => {
           this.style.color === 'red'
           ? this.style.color = 'blue':
           this.style.color = 'red';
      });
    }
  connectedCallback() {
  /*called when the element is
    connected to the page
  */
    this.style.color = 'blue';

    const template =
    document.querySelector('template');

    const clone = document.importNode(template.content, true);

  //this.appendChild(clone);
    this.attachShadow({ mode: 'open' });
    this.shadowRoot.appendChild(clone);
   }
 }
 customElements.define('my-component', MyComponent);
</script>
</body>
</html>
```

- shadow DOM의 자식 요소들은 외부에서 자식 요소로 간주되지 않는다.

## 동적으로 마이크로 프런트엔드 구성

### 마이크로 프런트엔드 레지스트리

- 다양한 스크립트의 지연된 로딩을 위해 중앙 웹 컴포넌트 레지스트리를 사용할 수 있지만 간단하게는 마이크로 프런트엔드 레지스트리로 시작할 수 있다.
- 로더 자바스크립트에서 동적으로 가져오고 로드하여 마이크로프런트엔드와 웹컴포넌트도 동적으로 제공한다.
- 마이크로 프런트엔드는 웹서버, 클라이언트의 런타임 구간에서 쉽게 패치할 수 있다.

### 런타임에 마이크로 프런트엔드 업데이트하기

- 마이크로 프런트엔드는 업데이트 경로를 단순화하는데, 전체 웹 서버나 어플리케이션을 다시 시작할 필요 없이 단일 모듈만 다시 시작할 수 있다.
- 웹 컴포넌트는 런타임 패치에 최적은 아니기 때문에 핫 리로드를 할 수 있다.

## 요약

- 클라이언트측 구성은 컨텐츠와 독립적으로 실행디는 몇가지 위젯과 함께 제공되는 웹 어플리케이션에 적용할 수 있어 대화형 그래픽에서 추가 정보를 찾을 수 있는 기사를 게시하는 온라인 신문에는 클라이언트측 구성이 적합하다.
