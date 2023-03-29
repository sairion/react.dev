---
title: "내장 React API들"
---

<Intro>

[Hooks](/reference/react)와 [Components](/reference/react/components) 외에도, `react` 패키지는 컴포넌트 정의에 유용한 몇 가지 여러 API들을 export합니다. 이 페이지에서는 나머지 모든 모던 React API를 나열합니다.

</Intro>

---

* [`createContext`](/reference/react/createContext)은 자식 컴포넌트에 context를 정의하고 제공할 수 있게 합니다. [`useContext`](/reference/react/useContext)와 함께 사용됩니다.
* [`forwardRef`](/reference/react/forwardRef)는 컴포넌트가 DOM 노드를 부모에게 ref로 노출하게 합니다. [`useRef`](/reference/react/useRef)와 함께 사용됩니다.
* [`lazy`](/reference/react/lazy)는 컴포넌트의 코드를 렌더링되기 전까지 로딩을 지연시킬 수 있습니다.
* [`memo`](/reference/react/memo)는 같은 props를 받아 다시 렌더링될 필요가 없는 경우 건너뛸 수 있게 합니다. [`useMemo`](/reference/react/useMemo)와 [`useCallback`](/reference/react/useCallback)와 함께 사용됩니다.
* [`startTransition`](/reference/react/startTransition)은 상태 업데이트를 우선순위가 높지 않음으로 표시할 수 있게 합니다. [`useTransition`](/reference/react/useTransition)과 유사합니다.
