# 마이크로 프론트엔드에서의 클라이언트 측 구성

## 클라이언트 측 구성

- 브라우저 상에서 여러 마이크로 프론트엔드를 통합하는 방식이다.
- js 기반의 프레임 워크를 사용해 iframe이나 DOM의 특정 요소에 렌더링한다.

### 👉🏻 App Shell(Application Shell)

- 스크립트 참조를 표현하는 주요 HTML 문서
- 마이크로 프런트엔드간의 상호 작용을 담당한다.
- 전역 종속성 구성
- 사용자 상호 작용을 기반으로 시기 결정

```html
<html>
  <head>
    <script src="app-shell-1.js"></script>
    <script src="app-shell-2.js"></script>
    <script src="app-shell-3.js"></script>
  </head>
</html>

```

```jsx
<Routes>
  <Route path="/" element={<h2>Welcome to the App Shell</h2>} />
  <Route path="/app-shell-1" element={<AppShell1 />} />
  <Route path="/app-shell-2" element={<AppShell2 />} />
  <Route path="/app-shell-2" element={<AppShell3 />} />
</Routes>
```

## 장단점

### 👉🏻 장점

- 유연성이 높다.
- 각 프론트엔드간의 독립성이 높다.

### 👉🏻 단점

- 자바스크립트에 의존하여 성능 문제와 접근성 문제가 발생한다.
- 초기 로드 속도가 느려질 수 있다.

## 웹 컴포넌트 살펴보기

- 사용자가 정의한 커스텀 요소를 만들 수 있다.

    ```jsx
    class MyButton extends HTMLElement {
      constructor() {
        super();
        
       const button = document.createElement('button');
      
        button.textContent = 'Click Me!';
        
        this.appendChild(button);
    }
    
    customElements.define('my-button', MyButton);
    ```

    ```html
    <body>
      <my-button></my-button>
    </body>
    ```

- Shadow DOM을 통해 DOM 트리 캡슐화할 수 있다.
- 템플릿 태그를 통해 재사용 가능한 DOM 트리를 만들 수 있다.

    ```html
    <template id="my-template">
     <style>
      p { background: tomato; }
     </style>
     <p>토마토 공간</p>
    </template>
    ```

    ```jsx
    const myTemplate = document.getElementById('my-template');
    const cloneElement = document.importNode(myTemplate.content, true);
    
    document.body.appendChild(cloneElement);
    ```

### 👉🏻 Shadow DOM

 HTML 요소의 **캡슐화된 DOM 트리를 만들어주는 브라우저 기술**로 요소의 내부 구현을 외부로부터 보호하거나 독립된 스타일 관리를 가능할 수 있게 한다.

- open: js를 사용하여 shadow DOM에 접근 가능
- closed: 외부로 부터 접근 불가

```jsx
const target = document.querySelector('#target');
const shadowRoot = elementRef.attachShadow({ mode: "open" });

// 스타일 격리
shadowRoot.innerHTML = `
 <style>
  p { background: tomato; }
 </style>
 <p>토마토 공간</p>
`
```

## 동적으로 마이크로 프런트엔드 구성

 마이크로 프런트엔드가 필요에 따라 로드되거나 업데이트가 가능해야 한다.

### 👉🏻 마이크로 프론트엔드 레지스트리 사용하기

마이크로 프론트엔드를 중앙에서 관리하는 시스템으로 이를 통해 동적 로딩과 업데이트가 가능해진다.

- 마이크로 프런트엔드의 위치, 버전 등을 정의한 메타데이터를 하나로 관리한다.
- 이 정보를 기반으로, App Shell은 각 마이크로 프론트엔드를 동적으로 로드할 수 있다.

### 👉🏻 런타임에 마이크로 프론트엔드 업데이트하기

 런타임에 사용자 행동에 대한 요구사항에 맞게 업데이트가 가능해야한다.

## 용어 정리

- orchestration: 여러 마이크로 프론트엔드륾 묶어 하나의 서비스를 만드는 개념. MFA에서는 마이크로 프론트엔드간의 데이터 전달, 이벤트 처리, 페이지 구성 등을 책임지는 매커니즘이다.
- hot reload: 새로고침 없이 컴포넌트 변경

## 참고 자료

- [**마이크로 프런트엔드 마이그레이션 여정 - 1부: 설계**](https://hackernoon.com/lang/ko/%EB%A7%88%EC%9D%B4%ED%81%AC%EB%A1%9C-%ED%94%84%EB%A1%A0%ED%8A%B8%EC%97%94%EB%93%9C-%EB%A7%88%EC%9D%B4%EA%B7%B8%EB%A0%88%EC%9D%B4%EC%85%98-1%EB%B6%80-%EB%94%94%EC%9E%90%EC%9D%B8%EC%9C%BC%EB%A1%9C%EC%9D%98-%EC%97%AC%ED%96%89)
- [**MDN - shadow DOM 사용하기**](https://developer.mozilla.org/ko/docs/Web/API/Web_components/Using_shadow_DOM)
