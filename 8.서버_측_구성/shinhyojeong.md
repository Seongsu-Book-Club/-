# 마이크로 프론트엔드에서의 서버측 구성

## 서버측 구성 방식의 특징

- 규모 변경성에 대한 제약 없이 독립된 상태를 유지하는 것에 최적화된 구성 방식 중 하나이다.
- 다양한 소스의 뷰를 동적으로 결합할 수 있고 공유 저장소가 필요하지 않아 수평적 마이크로 프론트엔드가 가능하다.
-

### 👉🏻 프록시, 집계 계층, BFF 또는 게이트 웨이

- 실제로 화면에 렌더링을 하기 위해 서로 다른 파일들을 하나로 연결해주는 역할을 한다.
- 여러가지 일이 일어나는데 URI로 식별되는 현재 페이지를 템플릿과 연결짓는 역할이 가장 중요하다. 예를들면

  ```
  ...
  <esi:include src="/mf-blue/buy-button" />
  ...
  ```

  위 코드가 어떤 화면과 연결할지 결정한다.

### 👉🏻 주의할 점

- 프록시를 통한 요청은 복원력이 뛰어나며 애플리케이션과 충돌하지 않도록 해야한다. 충분히 독립적으로 동작하여 마이크로프론트엔드를 개별적으로 동작하도록 해야한다.
- 사용자가 요청한 페이지와 렌더링하는 페이지가 일치하도록 해야한다.
- 하이퍼 참조 조정시 이슈를 피하기 위해 프록시 요청에 HTML이 포함되어있다면 응답을 파싱하여 검사해야한다.

## 장/단점

### 👉🏻 장점

- 규모에 크게 구애받지 않는다.
- 느슨한 결합과 높은 성능을 가진다.
- 독립적인 개발이 가능하며 유연성이 뛰어나다.

### 👉🏻 단점

- SPA의 경우에는 적절한 격리를 보장할 수 없다.
- 적절한 디버깅 툴이 없어 충돌을 잡는 것이 어렵다.

## 구성 레이아웃 만들기

### 👉🏻 레이아웃의 이해

- 서로 다른 마이크로 프론트엔드에서 개별 프레그먼트를 어디에 배치할지 정한다.
- 게이트웨이 서비스만이 클라이언트로 다시 보낼 수 있는 일부 HTML을 생성하기 위해 레이아웃을 적절하게 매치할 수 있다. (게이트웨이 서비스 === 레이아웃 서비스 === 레이아웃 엔진)
- 종류
  - 일반 레이아웃: 게이트웨이 서비스에 의해 정해진다.(예: 머릿글, 바닥글, 탐색 모음)
  - 특정 레이아웃: 개별 마이크로 프론트엔드에 의해 정해진다.(예: 내용)

### 👉🏻 SSI(Server Side Include) 사용하기

- 환경에 구애가 적은 편이며, 심지어 SSI를 지원하지 않는 웹 서버에서도 (빈 화면이겠지만)사용이 가능하다.
- HTML 주석을 사용하며 HTML 외에는 아무것도 필요하지 않다.
- 레이아웃은 데이터 베이스에 저장되며 생성/업데이트시에 관리할 수 있는 기능(CMS)을 제공하기 때문에 생산성이 좋다.

```HTML
<!-- #include virtual="footer" -->
```

### 👉🏻 ESI(Edge Side Include) 사용하기

- XML 기반의 ESI 태그를 사용한다. SSI에 비해 현대적이다.
- 보다 많은 기능을 제공하며 오류 처리도 가능하다.
- 구현하기가 복잡한 편이다.
- SSI와 동일하게 데이터베이스에 저장되며 생성/업데이트를 할 수 있다.

```HTML
<esi:include src="/footer" alt="/empty" onerror="continue" />
```

### 👉🏻 JS 템플릿 문자열 사용하기

- 유연하게 작성이 가능하고 코드를 최소한으로 작성할 수 있게 한다.
- 전체 서비스를 업데이트 하는 경우에만 업데이트가 가능 하다. 개발자가 배포하고 유지관리 해야하기 때문에 빠른 반복/개선이 어렵다.

```javascript
const name = "JS Template";
const content = `using ${name} example`;
```

## 생각 정리와 이야기 거리

- 책에서 설명하는 것처럼 SPA에서는 독립성을 강조하는 서버측 구성이 완전한 격리가 어렵기 때문에 현대의 방식과는 동떨어져있다는 생각이 들었다.
- 가볍게 읽어만 보는 것이 좋을 것 같은 챕터인듯 !
