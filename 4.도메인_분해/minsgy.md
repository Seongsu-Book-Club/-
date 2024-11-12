
# 마이크로프론트엔드에서의 관심사 분리

- [마이크로프론트엔드에서의 관심사 분리](#마이크로프론트엔드에서의-관심사-분리)
- [도메인 주도 설계의 원칙](#도메인-주도-설계의-원칙)
- [관심사 분리](#관심사-분리)
  - [기술적 분할](#기술적-분할)
  - [기능적 분할](#기능적-분할)
  - [생각한 선택 기준](#생각한-선택-기준)
  - [실제 예시 구조를 작업해보자](#실제-예시-구조를-작업해보자)
- [번외. 테스트 코드](#번외-테스트-코드)
  - [테스트 - 기술적 분할](#테스트---기술적-분할)
  - [테스트 - 기능적 분할](#테스트---기능적-분할)
  - [결론](#결론)

---

# 도메인 주도 설계의 원칙

1. DDD의 핵심 개념
    - 도메인 주도 설계는 복잡한 소프트웨어를 도메인(업무 영역) 중심으로 설계하는 방법론입니다
    - 비즈니스 도메인을 중심으로 설계하여 기술보다 도메인 모델에 집중합니다
    - 도메인 전문가와 개발자 간의 공통된 언어(Ubiquitous Language)를 사용합니다

2. 핵심 구성요소

    A) Module (모듈)

    - 연관된 도메인 개념들을 하나의 단위로 묶은 것
    - **높은 응집도와 낮은 결합도**를 지향
    - 마이크로프론트엔드에서는 독립적으로 배포/운영 가능한 UI 컴포넌트 단위로 구현됨

    B) Bounded Context (경계가 있는 컨텍스트)

    - 특정 도메인 모델이 적용되는 명확한 경계
    - 각 컨텍스트 내에서는 일관된 용어와 규칙이 적용됨
    - 마이크로프론트엔드 적용
        - 각 팀이 담당하는 독립적인 프론트엔드 애플리케이션
        - 자체적인 기술 스택과 배포 주기를 가질 수 있음
        - 명확한 인터페이스를 통해 다른 컨텍스트와 통신

    C) Context Map (컨텍스트 맵)

    - 여러 Bounded Context 간의 관계를 시각화/문서화
    - 통합 지점과 변환 규칙을 정의
    - 마이크로프론트엔드 적용
        - 각 프론트엔드 앱 간의 통신 방식 정의
        - 공유 데이터/상태 관리 전략
        - 라우팅 및 네비게이션 규칙
3. 마이크로 프론트엔드와의 연관성
    - 각 기능 영역을 독립적인 프론트엔드 애플리케이션으로 분리
    - 팀 간 자율성과 확장성 확보
    - 공통 구성요소
        - 모듈화 된 설계
        - 명확한 경계 정의
        - 표준화된 통신 방식

# 관심사 분리

## 기술적 분할

특징

- 화면 구성 요소를 그룹화하여 서로 다른 마이크로 프런트엔드 배치
- 프로그램의 각 부분이 단일한 목적과 책임이 가지도록 분리하는 원칙
- 복잡성 관리, 유지보수성 향상, 재사용성 증가 주요 목표

장점

- 기술 스택별 명확한 경계
- 기술 변경 시 해당 레이어만 수정
- 전문성에 따른 팀 구성 용이

단점

- 기능 변경 시 여러 레이어 수정 필요
- 레이어 간 의존성 관리 필요
- 과도한 추상화 가능성

## 기능적 분할

특징

- 비즈니스/도메인 기준으로 분할
- 각 모듈은 특정 비즈니스 기능을 완전히 담당함
- 수평적인 구조로 구성

장점

- 기능 변경 시 해당 모듈만 수정하기 용이하다
- 도메인에 대한 이해도가 향상된다
- 독립적인 배포와 확장이 용이하다

단점

- 코드 중복의 가능성이 높음
- 공통 기능 관리가 필요로 하다. 중복되는 기능들이 많을 수 있어서
- 기술에 대한 표준화를 잡기가 모호해진다

위에 대한 설계에 대한 디자인 패턴인 **FSD(Feature Sliced Design)**가 있음

## 생각한 선택 기준

기술적 분할

- 작은 규모의 프로젝트에 적합
- 기술 스택의 변경이 잦을 경우 구성
- 기획 변경이 최소화되어 있는 프로젝트일 경우
- 개발자 전문성 기반의 팀 구성
  - 걱정되는 점은 온보딩이 쉬워보이지 않는다

기능적 분할

- 큰 규모의 프로젝트
- 활발하게 변경되는 기능 단위들
  - 추상화가 오히려 개발에 발목을 잡는 상황일 때
- 마이크로 프론트엔드 아키텍처를 가져갈 때 분리해서 작업하기 간편
- 독립적인 팀 운영을 위할 때 나눠 작업하기 유용하다

## 실제 예시 구조를 작업해보자

```jsx
// 1. 기술적 분할 (Technical Split) 구조
src/
├── components/              # UI 컴포넌트 레이어
│   ├── common/              # 공통 컴포넌트
│   │   ├── Button.tsx
│   │   ├── Input.tsx
│   │   ├── Modal.tsx
│   │   └── Card.tsx
│   ├── products/            # 상품 관련 컴포넌트
│   │   ├── ProductCard.tsx
│   │   ├── ProductList.tsx
│   │   ├── ProductDetail.tsx
│   │   └── ProductFilter.tsx
│   ├── cart/               # 장바구니 컴포넌트
│   │   ├── CartItem.tsx
│   │   ├── CartList.tsx
│   │   └── CartSummary.tsx
│   └── checkout/           # 결제 컴포넌트
│       ├── CheckoutForm.tsx
│       └── PaymentMethod.tsx
│
├── hooks/                  # 비즈니스 로직 레이어
│   ├── useProducts.ts
│   ├── useCart.ts
│   ├── useOrder.ts
│   └── useAuth.ts
│
├── services/              # 데이터 액세스 레이어
│   ├── api.ts             # API 클라이언트 설정
│   ├── productService.ts
│   ├── cartService.ts
│   ├── orderService.ts
│   └── authService.ts
│
├── store/               # 상태 관리 레이어
│   ├── products/
│   │   ├── productsSlice.ts
│   │   └── productsSelectors.ts
│   ├── cart/
│   │   ├── cartSlice.ts
│   │   └── cartSelectors.ts
│   └── store.ts
│
├── types/               # 타입 정의
│   ├── product.ts
│   ├── cart.ts
│   ├── order.ts
│   └── user.ts
│
└── utils/              # 유틸리티 함수
    ├── format.ts
    ├── validation.ts
    └── helpers.ts
```

```jsx
// 2. 기능적 분할 (Functional Split) 구조
src/
├── features/          # 기능별 모듈
│   ├── product-catalog/     # 상품 카탈로그 기능
│   │   ├── components/     # 기능별 컴포넌트
│   │   │   ├── ProductCard.tsx
│   │   │   ├── ProductList.tsx
│   │   │   ├── ProductDetail.tsx
│   │   │   └── ProductFilter.tsx
│   │   ├── hooks/         # 기능별 훅
│   │   │   ├── useProductList.ts
│   │   │   └── useProductDetail.ts
│   │   ├── services/      # 기능별 서비스
│   │   │   └── productService.ts
│   │   ├── store/         # 기능별 상태 관리
│   │   │   ├── productSlice.ts
│   │   │   └── productSelectors.ts
│   │   └── types/         # 기능별 타입
│   │       └── product.ts
│   │
│   ├── shopping-cart/      # 장바구니 기능
│   │   ├── components/
│   │   │   ├── CartItem.tsx
│   │   │   ├── CartList.tsx
│   │   │   └── CartSummary.tsx
│   │   ├── hooks/
│   │   │   └── useCart.ts
│   │   ├── services/
│   │   │   └── cartService.ts
│   │   └── store/
│   │       └── cartSlice.ts
│   │
│   └── checkout/          # 결제 기능
│       ├── components/
│       │   ├── CheckoutForm.tsx
│       │   └── PaymentMethod.tsx
│       ├── hooks/
│       │   └── useCheckout.ts
│       ├── services/
│       │   └── checkoutService.ts
│       └── store/
│           └── checkoutSlice.ts
│
├── shared/           # 공통 모듈
│   ├── components/   # 공통 컴포넌트
│   │   ├── Button.tsx
│   │   ├── Input.tsx
│   │   └── Modal.tsx
│   ├── hooks/       # 공통 훅
│   │   ├── useAuth.ts
│   │   └── useForm.ts
│   ├── services/    # 공통 서비스
│   │   └── api.ts
│   └── utils/       # 공통 유틸리티
│       ├── format.ts
│       └── validation.ts
│
└── core/           # 핵심 설정
    ├── config/    # 앱 설정
    ├── router/    # 라우팅 설정
    └── store/     # 전역 상태 설정

```

# 번외. 테스트 코드

기술적 분할 테스트 방법과 기능적 분할의 통합테스트에 대하여 정리해보았습니다

## 테스트 - 기술적 분할

장점

- 레이어 별 테스트가 용이하며 테스트 패턴에 일관성을 가질 수 있다.
- 모킹을 하기 위한 데이터가 명확하게 있다
- 명확한 의존성을 파악할 수 있다.

단점

- 통합 테스트와 기능 흐름 테스트가 복잡하고 구성하기 어렵다
- 여러 레이어를 걸치는 테스트 구성이 어렵고 복잡

```tsx
// components/__tests__/ProductCard.test.tsx
describe('ProductCard Component', () => {
  it('renders product information correctly', () => {
    const product = {
      id: '1',
      name: 'Test Product',
      price: 100
    };
    render(<ProductCard product={product} />);
    expect(screen.getByText('Test Product')).toBeInTheDocument();
  });
});

// services/__tests__/productService.test.ts
describe('Product Service', () => {
  it('fetches products correctly', async () => {
    const products = await productService.getProducts();
    expect(products).toHaveLength(10);
  });
});
```

## 테스트 - 기능적 분할

장점

- 기능 단위의 격리 된 테스트 환경
- 테스트 구성이 직관적이다
- 비즈니스 로직 중심의 테스트가 용이하다
- 사용자 기반의 시나리오 테스트가 용이하다

단점

- 중복되는 테스트 코드가 발생할 수 있고 공통 기능 테스트 관리가 필요로 한다
- 의존성 모킹이 복잡할 수도 있다

```tsx
// features/product-catalog/__tests__/integration/productCatalog.test.tsx
describe('Product Catalog Feature', () => {
  it('shows filtered products when filter is applied', async () => {
    render(<ProductCatalog />);
    // 필터 적용
    fireEvent.click(screen.getByText('Filter'));
    // 결과 확인
    await waitFor(() => {
      expect(screen.getByTestId('product-list')).toMatchSnapshot();
    });
  });
});

// features/product-catalog/__tests__/unit/useProductCatalog.test.ts
describe('useProductCatalog Hook', () => {
  it('handles filter changes correctly', () => {
    const { result } = renderHook(() => useProductCatalog());
    act(() => {
      result.current.setFilters({ category: 'electronics' });
    });
    expect(result.current.products).toMatchSnapshot();
  });
});
```

## 결론

결론적으로 통합테스트 시 **기능적 분할** 방법이 효율적이고 구성하기 좋아보인다.

이유는

- 비즈니스 시나리오 중심 테스트
  - 실제 사용자 흐름과 유사한 테스트 작성
  - 기능 요구사항 검증이 직관적으로 나타남
- 테스트 격리성
  - 각 기능 별 독립적인 테스트 환경
  - 테스트간 간섭이 최소화되어 있음
- 유지보수성
  - 기능 변경 시 관련 테스트만 수정
  - 낮은 결합도와 높은 응집도
