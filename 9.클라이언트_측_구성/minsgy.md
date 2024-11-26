# 클라이언트 측 구성의 기본

클라이언트 측 구성은 브라우저에서 직접 마이크로 프론트엔드 애플리케이션을 통합하는 방식

## 주요 특징

- JavaScript를 사용하여 런타임에 각 마이크로 프론트엔드를 로드하고 마운트합니다.
- 브라우저에서 직접 통합이 이루어지므로 서버 의존성이 적습니다.
- 각 팀이 독립적으로 개발하고 배포할 수 있습니다.

## 구현 방식

Script 태그를 통한 로딩

```jsx

<script src="team1-app.js"></script>
<script src="team2-app.js"></script>

```

전역 함수나 이벤트를 통한 통신

```jsx

window.renderApp1('#container1');
window.renderApp2('#container2');

```

## 장/단점

장점

- 유연한 구성이 가능합니다
- 서버 의존성이 낮아 확장성이 좋습니다
- 동적 로딩으로 초기 로드 시간을 최적화할 수 있습니다
- 각 팀이 독립적인 개발 사이클을 가질 수 있습니다

단점

- 초기 페이지 로딩 시간이 증가할 수 있습니다
- JavaScript 번들 크기가 커질 수 있습니다
- 클라이언트 리소스 사용량이 증가합니다
- 중복 코드나 의존성 문제가 발생할 수 있습니다

## 3. 웹 컴포넌트 살피기

웹 컴포넌트는 재사용 가능한 커스텀 엘리먼트를 만들 수 있게 해주는 웹 표준 기술입니다

Custom Elements

```jsx
class MyComponent extends HTMLElement {
  connectedCallback() {
    this.innerHTML = '<h1>My Component</h1>';
  }
}
customElements.define('my-component', MyComponent);

```

Shadow DOM

```jsx
class MyComponent extends HTMLElement {
  constructor() {
    super();
    const shadow = this.attachShadow({mode: 'open'});
    shadow.innerHTML = '<style>h1 { color: red; }</style><h1>Hello!</h1>';
  }
}

```

HTML Templates

```html
<template id="my-template">
  <style>
    .container { padding: 20px; }
  </style>
  <div class="container">
    <slot></slot>
  </div>
</template>

```

## 4. 동적으로 마이크로 프론트엔드 구성

동적 구성은 필요한 시점에 마이크로 프론트엔드를 로드하고 마운트하는 방식입니다

1. 동적 스크립트 로딩

```jsx

function loadMicroFrontend(url) {
  const script = document.createElement('script');
  script.src = url;
  document.head.appendChild(script);
}

```

1. SystemJS나 Module Federation 사용

```jsx
System.import('app1').then(module => {
  module.mount('#app1-container');
});

```

1. 라우팅 기반 로딩

```jsx

const routes = {
  '/app1': () => loadMicroFrontend('app1'),
  '/app2': () => loadMicroFrontend('app2')
};

```

성능 최적화

- 레이지 로딩 구현
- 프리로딩 전략 사용
- 캐싱 메커니즘 도입
- 번들 크기 최적화

모니터링과 에러 처리

```jsx
window.addEventListener('error', (event) => {
  console.error('Micro-frontend error:', event.error);
 // 폴백 UI 표시 또는 에러 리포팅 
  // onload, onerror 를 활용해도 된다.

});

```

## 번외. 전역 함수 및 이벤트 통신 케이스 정리하기

### 1. 전역 함수를 통한 통신

가장 기본적인 방식으로 window 객체에 함수를 등록하여 사용합니다

```jsx

// App1에서 전역 함수 정의
window.app1 = {
  updateData: function(data) {
    console.log('App1 received:', data);
// App1의 상태 업데이트 로직
  },
  getData: function() {
    return this.currentState;
  }
};

// App2에서 App1의 함수 호출
window.app1.updateData({ message: 'Hello from App2' });

```

### 2. Custom Events를 통한 통신

CustomEvent를 사용하여 마이크로 프론트엔드 간 통신을 구현합니다

```jsx

// 이벤트 발신
function sendEvent(eventName, data) {
  const event = new CustomEvent(eventName, {
    detail: data,
    bubbles: true
  });
  window.dispatchEvent(event);
}

// 이벤트 수신
window.addEventListener('userUpdated', (event) => {
  console.log('Received user data:', event.detail);
});

// 사용 예시
sendEvent('userUpdated', {
  id: 1,
  name: 'John'
});

```

### 3. MutationObserver를 통한 DOM 변경 감지

DOM 변경을 감지하여 마이크로 프론트엔드 간 상태를 동기화합니다

```jsx

// MutationObserver 설정
const observer = new MutationObserver((mutations) => {
  mutations.forEach((mutation) => {
    if (mutation.type === 'attributes') {
      const newValue = mutation.target.getAttribute(mutation.attributeName);
      console.log(`Attribute ${mutation.attributeName} changed to ${newValue}`);
      handleAttributeChange(mutation.attributeName, newValue);
    }
  });
});

// 관찰 시작
const targetNode = document.getElementById('shared-component');
observer.observe(targetNode, {
  attributes: true,
  childList: true,
  subtree: true
});

// 정리 함수
function cleanup() {
  observer.disconnect();
}

```

## 참고자료

- <https://www.youtube.com/watch?v=urdISeZ4l-s>
