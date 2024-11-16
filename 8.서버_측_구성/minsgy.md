# 마이크로 프론트엔드 기법 중 하나인, 서버 측 구성을 정리합니다. (Server Side Composition)

## 서버 측 구성 기본

서버에서 마이크로프론트엔드 조각들을 조합하여 완성된 페이지(HTML)를 클라이언트에 전달하는 구조로 개발하는 방법을 말합니다.

- 위와 같은 방법으로 각 팀마다 독립적으로 개발한 컴포넌트를 서버에 배포하는 방식으로 화면을 구성합니다.
- 중앙 서버(리버스 프록시 서버)를 두어 라우팅과 페이지 세팅을 관리합니다.
  - 리버스 프록시(Reverse Proxy)는 클라이언트와 서버 사이에서 요청을 중개하는 서버로, 클라이언트는 리버스 프록시를 실제 서버로 인식하게 됨. (로드밸런싱, 캐싱, 보안, 압축 등 중앙서버 역할 중)

## 장점

- 초기 페이지 로딩 속도가 빠르다 (SSR 기반으로 이루어져서) + SEO 까지
- 브라우저 메모리 최소화 사용 등 성능에서도 우수
  - 독립적인 개발로 수준 높은 유연성을 보장한다는데 잘모르겠음 → 스타일시트 충돌, 디버깅 도구 부족으로 오히려 확장하기도 어렵지 않을까라는 생각

## 단점

- 단일 페이지로 병합되는 구조로 병합되서 완벽한 분리는 어렵다. 이때문에 스크립트나 스타일시트 코드가 충돌할 수 있다
  - 추가적인 검증 과정이 필요할 수 있다
- 서버 환경과 분리되어 있고 관련 된 디버깅 도구가 없어서 웹 사이트 디버깅하기가 어렵다.
  - 위 문제를 해결하기 위해서 여러 서버 프레임워크를 사용할 수 있습니다. (Podium + Fastify)

## 구성 레이아웃 만들기

서버 측 구성은 게이트웨이 서비스(Reverse Proxy 역할)를 활용해 서로 다른 마이크로 프론트엔드 모듈들을 합치게 됩니다.

### 레이아웃 구조 분리하는 이유

레이아웃이라는 명칭과 같이 페이지를 구조화하여 각 분리된 모듈들을 개별 Fragment를 어디에 배치할 지 고민하고 구조화 관점을 도입하기 위해 기술들을 도입합니다.

- 관련 기술로는 SSI/ESI/JS Template 등 여럿 존재합니다

### ESI(Edge Side Include)

- CDN과 같이 Edge 계층에서 컨텐츠를 조립하고 사용되는 Markup 언어입니다.
  - 단순한 구성으로 작업할 수 있지만 그만큼 유연하지 못합니다
  - 변경에 대해서는 어려울 수 있어서 사용 사례를 잘 고려해야 합니다

```jsx
<html>
  <body>
    This is my fragment:{" "}
    <esi:include src="https://our.fragments.com/fragment.html" />
  </body>
</html>
```

## 정리

- 서버 측 구성을 통해 브라우저 과부화없이 빠른 사용자 경험을 제공할 수 있다는 점이 장점입니다.
  - SSR 장점 (JavaScript 로드 최소화 등)

## 번외1. Podium + Fastify 방식

- Fastify 플러그인을 활용한 Server Side 방식의 화면 세팅

```jsx
// Podium + Fastify 예시
const fastify = require("fastify")()
const { Layout } = require("@podium/podlet")
const podiumFastify = require("@podium/fastify")

// Podium 레이아웃 설정
const layout = new Layout({
  name: "myLayout",
  pathname: "/",
})

// Fastify에 Podium 플러그인 등록
fastify.register(podiumFastify, {
  layout: layout,
})

// 마이크로프론트엔드 등록
const headerPodlet = layout.client.register({
  name: "header",
  uri: "http://localhost:7100/manifest.json",
})

const productsPodlet = layout.client.register({
  name: "products",
  uri: "http://localhost:7200/manifest.json",
})

// 라우트 처리
fastify.get("/", async (request, reply) => {
  const incoming = request.podium

  // 각 마이크로프론트엔드 페치
  const [header, products] = await Promise.all([
    headerPodlet.fetch(incoming),
    productsPodlet.fetch(incoming),
  ])

  // HTML 조합
  const html = `
        <!DOCTYPE html>
        <html>
            <head>
                <title>My Store</title>
                ${header.css.map((css) => css.toHTML()).join("\n")}
                ${products.css.map((css) => css.toHTML()).join("\n")}
            </head>
            <body>
                ${header.content}
                ${products.content}
                ${header.js.map((js) => js.toHTML()).join("\n")}
                ${products.js.map((js) => js.toHTML()).join("\n")}
            </body>
        </html>
    `

  reply.type("text/html").send(html)
})
```
