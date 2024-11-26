## SPA 내에서의 의존성 공유

가장 쉽게 공유하는 방법은 앱 셸에서 공유하는 것.

다만, 웹팩을 사용하는 경우 externals를 사용해야 하는데, 이 경우 전역 변수가 오염된다. 차선책으로 ESM 시스템을 이용할 수 있지만, 구식 브라우저에서 문제가된다.

이를 대체하는 적절한 옵션은 SystemJS를 이용하는 방법이고, 다음과 같은 방식으로 앱 셸에 선언할 수 있다.

```javascript
function registerModules(modules) {
  Object.keys(modules).forEach((name) => registerModule(name, modules[name]));
}

function registerModule(name, resolve) {
  System.register(name, [], (_exports) => ({
    execute() {
      const content = resolve();

      if (content instanceof Promise) {
        return content.then((innerContent) => _exports(innerContent));
      } else {
        _exports(content);
      }
    },
  }));
}
```

import maps 옵션도 제공하는데, 이는 셸에 직접 번들링되지 않고, 마이크로 프론트엔드가 로드될때 적용이 가능하다.

```typescript
{
    "imports": {
        "react": "https://unpkg.com/react/umd/react.production.min.js",
        "react-dom": "https://unpkg.com/react-dom/umd/react-dom.production.min.js",
        "react-router-dom": "https://unpkg.com/react-router-dom/umd/react-router-dom.min.js"
    }
}
```

## 프레임워크간 컴포넌트 사용

single-spa에서의 Parcel은 재사용 가능한 UI 컴포넌트의 단위이고, 마이크로 프론트엔드의 컴포넌트 공유를 위해 사용이 가능하다.

크게 동기식 마운팅과 비동기식 마운팅이 존재한다.

```typescript
// Synchronous mounting
const parcel = singleSpa.mountRootParcel(parcelConfig, {
  prop1: "value1",
  domElement: document.getElementById("a-div"),
});

parcel.mountPromise.then(() => {
  console.log("finished mounting the parcel!");
});

// Asynchronous mounting. Feel free to use webpack code splits or SystemJS dynamic loading
const parcel2 = singleSpa.mountRootParcel(() => import("./some-parcel.js"), {
  prop1: "value1",
  domElement: document.getElementById("a-div"),
});
```

## 통신 패턴

Prop, Event 교환 방식 등이 존재.

### Prop 이용

```typescript
import { Catalog } from "@mfe/catalog";
import { Cart } from "@mfe/cart";

const App = () => {
  const [productsInCartCount, setProductsInCartCount] = useState(0);

  const addToCart = () => {
    setProductsInCartCount(productsInCartCount + 1);
  };

  return (
    <>
      <Catalog onAddToCart={addToCart} />
      <Cart productsCount={productsInCartCount} />
    </>
  );
};
```

장점은 다음과 같다.

1. 컴포넌트 기반 아키텍쳐에서 널리 알려진 대중적인 방법
2. 대부분의 프레임워크들이 지원

단점은 다음과 같다.

1. 마이크로 프론트엔드와 컨테이너간 커플링
2. 두 마이크로 프론트엔드가 동일한 프레임워크가 아닌 경우 구현이 어려움

### Event 교환

```typescript
// Catalog MFE
const Catalog = () => {
  // fetch initial count first
  const [productsInCartCount, setProductsInCartCount] = useState(
    initialCount || 0
  );

  const addToCart = () => {
    const addToCartEvent = new CustomEvent("ADD_TO_CART", {
      detail: { count: productsInCartCount + 1 },
    });
    window.dispatchEvent(addToCartEvent);
  };

  return <Product onAddToCart={addToCart} />;
};

// Cart MFE
const Cart = () => {
  // fetch initial count first
  const [productsInCartCount, setProductsInCartCount] = useState(
    initialCount || 0
  );

  useEffect(() => {
    const listener = ({ detail }) => {
      setProductsInCartCount(detail.count);
    };

    window.addEventListener("ADD_TO_CART", listener);

    return () => {
      event.unsubscribe("ADD_TO_CART", listener);
    };
  }, []);

  return <NumberOfProductsAdded count={productsInCartCount} />;
};
```

장점은 다음과 같다.

1. 브라우저 플랫폼 내장 기능 이용
2. 확장하기 쉬움
3. 모든 마이크로 프론트엔드 팀이 따를 수 있는 일반적인 방법

단점은 다음과 같다.

1. 모바일 마이크로 프론트엔드에서 사용이 불가
2. 초기 셋업이 어려움

## References

[5 Different Techniques for Cross Micro Frontend Communication | by Vishal Sharma | Medium](https://sharvishi9118.medium.com/cross-micro-frontend-communication-techniques-a10fedc11c59)

[Applications API | single-spa](https://single-spa.js.org/docs/api/#mountrootparcel)
