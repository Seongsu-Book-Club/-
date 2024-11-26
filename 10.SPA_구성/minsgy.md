# SPA 구성의 기본 사항

SPA는 단일 HTML 페이지에서 동적으로 콘텐츠를 업데이트하는 웹 애플리케이션입니다.

## 기본 구조

스크립트 코드를 기반하여 동적인 페이지를 구성할 수 있도록 구성한 방식

```jsx

// index.html
<!DOCTYPE html>
<html>
<head>
  <title>SPA Application</title>
</head>
<body>
  <div id="app"></div>
  <script src="app.js"></script>
</body>
</html>

```

## +라우팅 구현)

JavaScript 환경에서 SPA 페이지 라우팅 관리 예제코드

```jsx

const router = {
  init: function() {
    this.handleRoute(window.location.pathname);
    window.addEventListener('popstate', (e) => {
      this.handleRoute(window.location.pathname);
    });
  },

  handleRoute: function(path) {
    switch(path) {
      case '/':
        renderHome();
        break;
      case '/about':
        renderAbout();
        break;
      default:
        render404();
    }
  }
};

```

# 2. 장단점

## 장점

1. 향상된 사용자 경험
    - 페이지 새로고침 없이 빠른 전환
2. 효율적인 리소스 관리
    - 초기 로드 후 필요한 데이터만 전송
    - 서버 부하 감소
3. 프론트엔드/백엔드 분리
    - 독립적인 개발 가능
    - API 기반 통신

## 단점

1. 초기 로딩 시간
    - 전체 애플리케이션 리소스 다운로드
    - 번들 크기 관리 필요
2. SEO 문제
    - 검색 엔진 최적화 어려움
    - SSR 구현 필요성
3. 브라우저 히스토리 관리
    - 복잡한 라우팅 처리
    - 뒤로가기 버튼 처리

# 3. 핵심 SPA 셸 구축하기

## 기본 셸 구조

```jsx

// app-shell.js
class AppShell {
  constructor() {
    this.container = document.getElementById('app');
    this.initialize();
  }

  initialize() {
// 공통 컴포넌트 렌더링
    this.renderHeader();
    this.renderNavigation();
    this.renderMainContent();
    this.renderFooter();
  }

  renderHeader() {
    const header = document.createElement('header');
    header.innerHTML = `
      <nav>
        <a href="/">Home</a>
        <a href="/about">About</a>
      </nav>
    `;
    this.container.appendChild(header);
  }

	// 다른 렌더링 메서드들...
}

```

# 4. 통신 패턴

## 데이터 공유 패턴

- window 객체를 활용한 데이터 공유 예시

```jsx
// 데이터 공유 예시
const sharedState = window.__SHARED_STATE__;
sharedState.registerDomain('user', {
  currentUser: null,
  preferences: {}
});

sharedState.subscribe('user', (state) => {
  console.log('User state updated:', state);
});
```

- 컴포넌트 제공 방식 (+Loader)

```jsx

// content-loader.js
class ContentLoader {
  async loadModule(moduleName) {
    try {
      const module = await import(`./modules/${moduleName}.js`);
      return module.default;
    } catch (error) {
      console.error(`Failed to load module: ${moduleName}`, error);
      return this.loadErrorModule();
    }
  }

  async loadComponent(componentName) {
    const component = await this.loadModule(componentName);
    return component;
  }
}

// 컴포넌트 제공 예시
const registry = new ComponentRegistry();
const loader = new ContentLoader(registry);

// 컴포넌트 등록
registry.register('shopping', 'Cart', ShoppingCart, {
  version: '1.0.0',
  dependencies: ['user']
});

//...
```

---

## 번외. 다른 여러 패턴들

## 이벤트 기반 통신

DOM API를 활용할 수도 있고, 이를  기반한 EventBus 패턴

```jsx

// event-bus.js
class EventBus {
  constructor() {
    this.events = {};
  }

  on(event, callback) {
    if (!this.events[event]) {
      this.events[event] = [];
    }
    this.events[event].push(callback);
  }

  emit(event, data) {
    if (this.events[event]) {
      this.events[event].forEach(callback => callback(data));
    }
  }
}

// 사용 예시
const eventBus = new EventBus();
eventBus.on('userLoggedIn', (user) => {
  updateUI(user);
});

```

## 웹소켓 통신

```jsx

// websocket-service.js
class WebSocketService {
  constructor(url) {
    this.url = url;
    this.socket = null;
    this.reconnectAttempts = 0;
    this.maxReconnectAttempts = 5;
  }

  connect() {
    this.socket = new WebSocket(this.url);

    this.socket.onopen = () => {
      console.log('WebSocket Connected');
      this.reconnectAttempts = 0;
    };

    this.socket.onmessage = (event) => {
      const data = JSON.parse(event.data);
      this.handleMessage(data);
    };

    this.socket.onclose = () => {
      console.log('WebSocket Disconnected');
      this.reconnect();
    };
  }

  reconnect() {
    if (this.reconnectAttempts < this.maxReconnectAttempts) {
      this.reconnectAttempts++;
      setTimeout(() => this.connect(), 1000 * this.reconnectAttempts);
    }
  }

  send(data) {
    if (this.socket && this.socket.readyState === WebSocket.OPEN) {
      this.socket.send(JSON.stringify(data));
    }
  }
}

```
