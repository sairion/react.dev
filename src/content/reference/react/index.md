---
title: "내장 React 훅들"
---

<Intro>

*훅*은 컴포넌트들에서 다양한 React 기능을 사용할 수 있게 해줍니다. 내장된 훅을 사용하거나 조합하여 직접 만들 수 있습니다. 이 페이지에서는 React의 모든 내장 훅을 나열합니다.

</Intro>

---

## state 훅 {/*state-hooks*/}

*State*는 컴포넌트가 [사용자 입력과 같은 정보를 "기억"](/learn/state-a-components-memory)할 수 있게 해줍니다. 예를 들어, 폼 컴포넌트는 입력 값을 저장하기 위해 state를 사용할 수 있으며, 이미지 갤러리 컴포넌트는 선택한 이미지 인덱스를 저장하기 위해 state를 사용할 수 있습니다.

컴포넌트에 state를 추가하기 위해 다음 중 하나의 훅을 사용하세요:

* [`useState`](/reference/react/useState)는 직접 업데이트할 수 있는 상태 변수를 선언합니다.
* [`useReducer`](/reference/react/useReducer)는 [리듀서 함수](/learn/extracting-state-logic-into-a-reducer) 내에 업데이트 로직이 포함된 상태 변수를 선언합니다. 

```js
function ImageGallery() {
  const [index, setIndex] = useState(0);
  // ...
```

---

## Context 훅 {/*context-hooks*/}

*Context* 는 컴포넌트가 [props로 전달하지 않고 먼 부모로부터 정보를 받을 수 있게](/learn/passing-props-to-a-component) 해줍니다. 예를 들어, 앱의 최상위 컴포넌트는 하위의 모든 컴포넌트가 얼마나 깊이 존재하든, 현재의 UI 테마를 전달할 수 있습니다.

* [`useContext`](/reference/react/useContext) 는 컨텍스트를 읽고 구독합니다.

```js
function Button() {
  const theme = useContext(ThemeContext);
  // ...
```

---

## Ref 훅 {/*ref-hooks*/}

*Refs* 는 DOM 노드나 timeout ID와 같이 렌더링에 사용되지 않는 정보를 컴포넌트가 [들고 있을 수 있게 합니다.](/learn/referencing-values-with-refs) state와는 달리, ref를 업데이트해도 컴포넌트는 다시 렌더링되지 않습니다. Ref는 React 패러다임에서 "escape hatch"로 사용됩니다. 내장된 브라우저 API와 같은 비-React 시스템을 다룰 때 유용합니다.

* [`useRef`](/reference/react/useRef)는 ref를 선언합니다. 어떤 값이든 보유할 수 있지만, 대체로 DOM 노드를 지정하는 데 사용됩니다.
* [`useImperativeHandle`](/reference/react/useImperativeHandle)은 컴포넌트에 노출된 ref를 재정의할 수 있게 합니다. 이 기능은 거의 사용되지 않습니다.

```js
function Form() {
  const inputRef = useRef(null);
  // ...
```

---

## Effect 훅 {/*effect-hooks*/}

*Effect*는 컴포넌트가 [외부 시스템에 연결하고 동기화할 수 있게 합니다.](/learn/synchronizing-with-effects) '외부 시스템'에는 네트워크, 브라우저 DOM, 애니메이션, 다른 UI 라이브러리로 작성된 위젯들 및 기타 비-React 코드가 포함됩니다.

* [`useEffect`](/reference/react/useEffect)는 컴포넌트를 외부 시스템에 연결합니다.

```js
function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(roomId);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]);
  // ...
```

Effect는 리액트 패러다임에서 "escape hatch"로 사용됩니다. 애플리케이션의 데이터 흐름을 다루기 위해 Effect를 사용하지 마세요. [외부 시스템과 상호작용하지 않는 경우, 효과가 필요하지 않을 수 있습니다.](/learn/you-might-not-need-an-effect)

실행 시점의 차이가 있는 `useEffect`의 거의 사용되지 않는 두 가지 배리에이션이 있습니다:

* [`useLayoutEffect`](/reference/react/useLayoutEffect)는 브라우저가 화면을 다시 paint하기 전에 발생합니다. 여기에서 레이아웃을 측정할 수 있습니다.
* [`useInsertionEffect`](/reference/react/useInsertionEffect)는 React가 DOM을 변경하기 전에 발생합니다. 라이브러리는 여기에 동적 CSS를 삽입할 수 있습니다.

---

## 퍼포먼스 훅 {/*performance-hooks*/}

리렌더링 성능을 최적화하는 일반적인 방법은 불필요한 작업을 건너뛰는 것입니다. 예를 들어, 이전 렌더링 이후 데이터가 변경되지 않았다면 React에게 캐시된 계산을 재사용하거나 리렌더링을 건너뛰도록 지시할 수 있습니다.

계산을 건너뛰고 불필요한 리렌더링을 방지하려면 다음 중 하나의 훅을 사용하세요:

*   [`useMemo`](/reference/react/useMemo)는 비용이 많이 드는 계산의 결과를 캐시할 수 있습니다.
*   [`useCallback`](/reference/react/useCallback)은 최적화된 컴포넌트로 전달하기 전에 함수 정의를 캐시할 수 있습니다.

```js
function TodoList({ todos, tab, theme }) {
  const visibleTodos = useMemo(() => filterTodos(todos, tab), [todos, tab]);
  // ...
}
```

화면이 실제로 업데이트되어야 하므로 리렌더링을 건너뛸 수 없는 경우도 있습니다. 이 경우, 동기적이어야 하는 blocking 업데이트(input에 타이핑하는 것과 같이)와 사용자 인터페이스를 차단할 필요가 없는 non-blocking 업데이트(차트를 업데이트하는 것과 같이)를 분리함으로서 성능을 향상시킬 수 있습니다.

렌더링의 우선순위를 정하려면 다음 훅 중 하나를 사용하세요:

- [`useTransition`](/reference/react/useTransition)을 사용하면 상태 전환을 non-blocking으로 표시하고 다른 업데이트가 중단하도록 허용할 수 있습니다.
- [`useDeferredValue`](/reference/react/useDeferredValue)를 사용하면 중요하지 않은 부분적 UI 업데이트를 지연시키고 다른 부분이 먼저 업데이트되도록 할 수 있습니다.

---

## 그 외의 훅들 {/*other-hooks*/}

이러한 훅은 대부분 라이브러리 작성자에게 유용하며 일반적으로 애플리케이션 코드에서는 사용되지 않습니다.

*   [`useDebugValue`](/reference/react/useDebugValue)는 사용자 정의 훅에 대한 React DevTools에서 표시되는 레이블을 사용자 정의할 수 있게 합니다.
*   [`useId`](/reference/react/useId)는 컴포넌트가 자체적으로 고유한 ID를 연결하도록 합니다. 일반적으로 접근성 API와 함께 사용됩니다.
*   [`useSyncExternalStore`](/reference/react/useSyncExternalStore)는 컴포넌트가 외부 저장소를 구독할 수 있게 합니다.

---

## 당신만의 훅 만들기 {/*your-own-hooks*/}

JavaScript 함수를 이용, [커스텀 훅을 정의](/learn/reusing-logic-with-custom-hooks#extracting-your-own-custom-hook-from-a-component)할 수도 있습니다.
