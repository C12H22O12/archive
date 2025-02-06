# useReactTable
- [전문 깃허브 링크](https://github.com/TanStack/table/blob/main/packages/react-table/src/index.tsx)

## Renderable
``` ts
export type Renderable<TProps> = React.ReactNode | React.ComponentType<TProps>
```

- React Element, React Element[], String, Boolean, Number, 혹은 TProps를 props로 가지는 React 컴포넌트로 타입을 제한

### ReactNode
``` ts
type ReactNode =
  | ReactElement
  | string
  | number
  | Iterable<ReactNode>
  | ReactPortal
  | boolean
  | null
  | undefined;
```
- React가 렌더링할 수 있는 모든 요소를 나타내는 타입
  - React Element, React Element[], String, Boolean, Number
- 이에는 null 또는 undefined가 포함되지 않으므로, 이를 허용하기 위해서는 `React.ReactNode`를 이용해야 함

#### 예시
```ts
<div> // <- ReactElement
  <Component> // <- ReactElement
    {condition && 'text'} // <- ReactNode
  </Component>
</div>
```

### ComponentType
- 해당 타입이 선언된 React 컴포넌트는 제네릭으로 받는 P를 props로 받도록 고정

``` ts
type ComponentType<P = {}> = ComponentClass<P> | FunctionComponent<P>;

interface FunctionComponent<P = {}> {
  (props: P, context?: any): ReactElement<any, any> | null;
  propTypes?: WeakValidationMap<P> | undefined;
  contextTypes?: ValidationMap<any> | undefined;
  defaultProps?: Partial<P> | undefined;
  displayName?: string | undefined;
}

interface ReactElement<
  P = any,
  T extends string | JSXElementConstructor<any> =
    | string
    | JSXElementConstructor<any>,
> {
  type: T;
  props: P;
  key: Key | null;
}
```

## isClassComponent
``` ts
function isClassComponent(component: any) {
  return (
    typeof component === 'function' &&
    (() => {
      const proto = Object.getPrototypeOf(component)
      return proto.prototype && proto.prototype.isReactComponent
    })()
  )
}
```
- component가 클래스 기반인지 확인
  - 클래스 컴포넌트일 경우 타입은 function(`typeof component === 'function'`)
  - component의 프로토타입을 통해 다음 내용 검사(`Object.getPrototypeOf(component)`)
    1. prototype이 존재하는가? (`proto.prototype`)
        - 클래스일 경우 참
    2. 리액트 컴포넌트인가? (`proto.prototype.isReactComponent`)
        - 리액트 클래스 컴포넌트일 경우 prototype 하위에 isReactComponent 속성 존재, 참으로 반환

## isExoticComponent
``` ts
function isExoticComponent(component: any) {
  return (
    typeof component === 'object' &&
    typeof component.$$typeof === 'symbol' &&
    ['react.memo', 'react.forward_ref'].includes(component.$$typeof.description)
  )
}
```

- component가 React.memo 또는 React.forwardRef로 감싸져있는 지 확인
- `component.$$typeof`을 통해 Symbol 값을 가짐으로써 요소의 타입을 식별

## isReactComponent
``` ts
function isReactComponent<TProps>(
  component: unknown
): component is React.ComponentType<TProps> {
  return (
    isClassComponent(component) ||
    typeof component === 'function' ||
    isExoticComponent(component)
  )
}
```

- component가 React컴포넌트인지 확인
- component가 리액트 클래스 컴포넌트거나, 함수이거나 특수 컴포넌트인지 확인
- 반환되는 조건이 참일 경우 component는 `React.ComponentType<TProps>`으로 취급

## flexRender
``` ts
/**
 * If rendering headers, cells, or footers with custom markup, use flexRender instead of `cell.getValue()` or `cell.renderValue()`.
 */
export function flexRender<TProps extends object>(
  Comp: Renderable<TProps>,
  props: TProps
): React.ReactNode | React.JSX.Element {
  return !Comp ? null : isReactComponent<TProps>(Comp) ? (
    <Comp {...props} />
  ) : (
    Comp
  )
}
```

- 컴포넌트가 명확한 값이 아니라면(undefined 혹은 null이라면) null을 반환
- 명확한 값이라면 리액트 컴포넌트인지 확인 후, 명제가 참이라면 파라미터 props를 Comp의 props로 전달. 거짓이면 Comp만 반환

## useReactTable
``` ts
export function useReactTable<TData extends RowData>(
  options: TableOptions<TData>
) {
  // Compose in the generic options to the user options
  const resolvedOptions: TableOptionsResolved<TData> = {
    state: {}, // Dummy state
    onStateChange: () => {}, // noop
    renderFallbackValue: null,
    ...options,
  }

  // Create a new table and store it in state
  // 테이블 객체를 저장
  // setState가 존재하지 않으니 불변 객체, 테이블 인스턴스를 유지
  const [tableRef] = React.useState(() => ({
    current: createTable<TData>(resolvedOptions),
  }))

  // By default, manage table state here using the table's initial state
  const [state, setState] = React.useState(() => tableRef.current.initialState)

  // Compose the default state above with any user state. This will allow the user
  // to only control a subset of the state if desired.
  tableRef.current.setOptions(prev => ({
    ...prev,
    ...options,
    state: {
      ...state,
      ...options.state,
    },
    // Similarly, we'll maintain both our internal state and any user-provided
    // state.
    onStateChange: updater => {
      setState(updater)
      options.onStateChange?.(updater)
    },
  }))

  return tableRef.current
}
```

- 옵션을 주입한 테이블 객체를 생성하고, 외부에서도 상태를 관리할 수 있도록 한 커스텀 훅
- `resolvedOptions` : 파라미터로 받은 Options에 state와 onStateChange, renderFallbackValue요소 추가
- `tableRef` : `resolvedOptions`이 반영된 테이블 객체 생성
- `[state, setState]` : 초기값을 `tableRef`의 초기값을 가지는 state. 이후 외부에서 객체의 값을 변경하면 함께 업데이트

### Q. 왜 tableRef에서 useState를 사용했을까?
``` ts
  // Create a new table and store it in state
  // 테이블 객체를 저장
  // setState가 존재하지 않으니 불변 객체, 테이블 인스턴스를 유지
  const [tableRef] = React.useState(() => ({
    current: createTable<TData>(resolvedOptions),
  }))
```

---
#### chatGPT 답변
💡 대체 가능 여부 → useRef도 가능하지만, useState를 선택한 이유가 있음

**✅ useRef를 사용한 경우**

useRef를 사용하면 .current 값을 변경해도 리렌더링이 발생하지 않음
하지만 useRef는 초기값을 함수로 전달해도 즉시 실행됨

**✅ useState를 사용한 이유**

useState의 초기값을 함수로 전달하면 최초 렌더링 시 한 번만 실행됨 (지연 실행)

즉, 불필요한 테이블 생성 연산을 방지할 수 있음
성능 최적화의 목적이 크며, 초기화 로직이 무거운 경우 useState가 유리

# 참고
- [React & TypeScript: What is ReactNode and when to use it? - Dev.to](https://dev.to/elhamnajeebullah/react-typescript-what-is-reactnode-and-when-to-use-it-3660)
- [When to use JSX.Element vs ReactNode vs ReactElement? - Stackoverflow](https://stackoverflow.com/questions/58123398/when-to-use-jsx-element-vs-reactnode-vs-reactelement)
- [React.ComponentType](http://fem-react-typescript.vercel.app/React.ComponentType.md)
- [Using type predicates](https://www.typescriptlang.org/docs/handbook/2/narrowing.html#using-type-predicates)
- [Why Do React Elements Have a $$typeof Property? - overreacted](https://overreacted.io/why-do-react-elements-have-typeof-property/)