---
title: useState
---

<Intro>

`useState`는 React Hook으로 컴포넌트에 [state 변수](/learn/state-a-components-memory)를 추가할 수 있게 해줍니다.

```js
const [state, setState] = useState(initialState);
```

</Intro>

<InlineToc />

---

## Reference {/*reference*/}

### `useState(initialState)` {/*usestate*/}

컴포넌트의 상단에서 `useState`를 호출하여 [state 변수](/learn/state-a-components-memory)를 선언합니다.

```js
import { useState } from 'react';

function MyComponent() {
  const [age, setAge] = useState(28);
  const [name, setName] = useState('Taylor');
  const [todos, setTodos] = useState(() => createTodos());
  // ...
```

state 변수는 `[something, setSomething]`와 같이 [array destructuring](https://javascript.info/destructuring-assignment)를 이용해 이름을 짓는 것이 컨벤션입니다.

[바로가기](#usage)에서 더 많은 예제를 확인할 수 있습니다.

#### 인자값 {/*parameters*/}

* `initialState`: state를 초기값으로 설정합니다. 어떤 타입의 값이든 초기값이 될 수 있지만, 함수의 경우 특별한 동작을 합니다. 이 인자는 초기 렌더링 이후에는 무시됩니다.
* `initialState`로 함수를 전달하면 _초기화 함수_ 로 취급됩니다. 이 함수는 순수해야 하며, 인자를 가져서는 안 되며, 어떤 타입이든 값을 반환해야 합니다. React는 컴포넌트를 초기화할 때 초기화 함수를 호출하고 반환 값을 최초 state로 저장합니다. [아래의 예제를 참조하세요.](#avoiding-recreating-the-initial-state)

#### 반환값 {/*returns*/}

`useState` 는 두 개의 값이 들어있는 배열을 반환합니다:

1.  현재의 state. 최초 렌더링 중에는 당신이 전달한 `initialState`와 일치합니다.
2.  [`set` 함수](#setstate)는 상태를 다른 값으로 업데이트하고 리렌더링을 할 수 있게 해줍니다.

#### 주의사항 {/*caveats*/}

* `useState`는 훅이므로 컴포넌트의 **최상위 레벨** 또는 자신의 훅에서만 호출할 수 있습니다. 루프 또는 조건문 내에서 호출할 수 없습니다. 필요한 경우 새 컴포넌트를 추출하고 상태를 해당 컴포넌트로 이동시킵니다.
* Strict 모드에서는 React가 [실수로 인한 impurities를 찾도록 도와주기 위해](#my-initializer-or-updater-function-runs-twice) 초기화 함수를 두 번 호출합니다. 이것은 개발 전용 동작으로, 프로덕션에는 영향을 미치지 않습니다. 초기화 함수가 순수 함수인 경우 동작에 영향을 주지 않아야 합니다. 두 호출 중 하나의


### `setSomething(nextState)` 와 같은 `set`함수 {/*setstate*/}

`useState`가 반환하는 `set` 함수를 사용하면 state를 다른 값으로 업데이트하고 리렌더링을 트리거할 수 있습니다. 다음 state를 직접 전달하거나 이전 state의 값으로부터 계산하는 함수를 전달할 수 있습니다:

```js
const [name, setName] = useState('Edward');

function handleClick() {
  setName('Taylor');
  setAge(a => a + 1);
  // ...
```

#### 매개변수 {/*setstate-parameters*/}

* `nextState`: state가 앞으로 되길 원하는 값입니다. 모든 타입의 값일 수 있지만 함수에 대한 특별한 동작이 있습니다.
  * 함수를 `nextState`에 전달하면 _업데이터 함수_ 로 취급됩니다. 이 함수는 순수해야 하고, 보류 중인 state의 값을 유일한 인수로 사용해야 하며, 다음 state를 반환해야 합니다. React는 업데이터 함수를 대기열에 넣고 컴포넌트를 다시 렌더링합니다. 다음 렌더링 중에 React는 대기열에 있는 모든 업데이터 함수를 이전 state에 적용하여 다음 state를 계산합니다. [아래 예시를 참조하세요.](#updating-state-based-on-the-previous-state)

#### 반환 값 {/*setstate-returns*/}

`set` 함수에는 반환 값이 없습니다.

#### 주의사항 {/*setstate-caveats*/}

* `set` 함수는 **다음* 렌더링에 대한 상태 변수만 업데이트합니다.** `set` 함수를 호출한 후 상태 변수를 읽으면, 호출 전 화면에 있던 [이전 값이 계속 표시됩니다.](#ive-updated-the-state-but-logging-gives-me-the-old-value)

* 만약 여러분이 제공한 새 값이 [`Object.is`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is) 비교에 의해 결정된 현재 `상태`와 동일하다면, React는 **컴포넌트와 그 children들의 리렌더링을 생략합니다**. 경우에 따라 React가 children들을 건너뛰기 전에 여전히 컴포넌트를 호출해야 할 수도 있지만, 코드에는 영향을 미치지 않아야 합니다.

* React는 [상태 업데이트를 일괄(batch) 처리합니다.](/learn/queueing-a-series-of-state-updates) **모든 이벤트 핸들러가 실행되고 난 후** `set` 함수들을 호출한 후에 화면을 업데이트합니다. 이렇게 하면 단일 이벤트 중에 여러 번 리렌더링하는 것을 방지할 수 있습니다. 드문 경우지만 DOM에 접근하기 위해 React가 화면을 더 일찍 업데이트하도록 강제해야 하는 경우, [`flushSync`.](/reference/react-dom/flushSync)를 사용할 수 있습니다.

* *렌더링 중* `set` 함수를 호출하는 것은 현재 렌더링 중인 컴포넌트 내에서만 허용됩니다. React는 반환값을 버리고 즉시 새로운 상태로 다시 렌더링을 시도합니다. 이 패턴은 거의 필요하지 않지만 **이전에 렌더링 된 정보를 저장**하는 데 사용할 수 있습니다. [아래 예시를 참조하세요.](#storing-information-from-previous-renders)

* 스트릭트 모드에서 React는 [실수로 발생한 impurities를 찾기 위해](#my-initializer-or-updater-function-runs-twice) **업데이터 함수를 두 번 호출**합니다. 이는 개발용 빌드에서만의 동작이며 프로덕션에는 영향을 미치지 않습니다. 업데이터 함수가 순수하다면(원래 그래야 하는 것처럼) 동작에 영향을 미치지 않습니다. 호출 중 하나의 결과는 무시됩니다.


---

## Usage {/*usage*/}

### 컴포넌트에 상태 추가하기 {/*adding-state-to-a-component*/}

컴포넌트의 최상위 레벨에서 `useState`를 호출하여 하나 이상의 [상태 변수](/learn/state-a-components-memory)를 선언하세요.

```js [[1, 4, "age"], [2, 4, "setAge"], [3, 4, "42"], [1, 5, "name"], [2, 5, "setName"], [3, 5, "'Taylor'"]]
import { useState } from 'react';

function MyComponent() {
  const [age, setAge] = useState(42);
  const [name, setName] = useState('Taylor');
  // ...
```

규칙은 [배열 구조 파괴]를 사용하여 `[something, setSomething]`과 같은 상태 변수의 이름을 지정하는 것입니다.

useState`는 정확히 두 개의 항목이 있는 배열을 반환합니다:

1. 이 상태 변수의 <CodeStep step={1}>현재 상태</CodeStep>, 처음에 입력한 <CodeStep step={3}>초기 상태</CodeStep>로 설정됩니다.
2. 상호작용에 따라 다른 값으로 변경할 수 있는 <CodeStep step={2}>`set` 함수</CodeStep>를 사용합니다.

화면의 내용을 업데이트하려면 다음 상태로 `set` 함수를 호출합니다:

```js [[2, 2, "setName"]]
function handleClick() {
  setName('Robin');
}
```

React는 다음 state를 저장하고 새로운 값으로 컴포넌트를 다시 렌더링한 후 UI를 업데이트합니다.

<Pitfall>

set` 함수를 호출하면 이미 실행 중인 코드에서 현재 상태가 변경되지 않습니다(#ive-updated-the-state-but-logging-gives-me-the-old-value):

```js {3}
function handleClick() {
  setName('Robin');
  console.log(name); // Still "Taylor"!
}
```

다음 렌더링부터 `useState`가 반환할 내용에만 영향을 줍니다.

</Pitfall>

<Recipes titleText="Basic useState examples" titleId="examples-basic">

#### 카운터 (숫자) {/*카운터-넘버*/}

이 예제에서 `count` 상태 변수는 숫자를 담고 있습니다. 버튼을 클릭하면 숫자가 증가합니다.

<Sandpack>

```js
import { useState } from 'react';

export default function Counter() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);
  }

  return (
    <button onClick={handleClick}>
      You pressed me {count} times
    </button>
  );
}
```

</Sandpack>

<Solution />

#### 텍스트 필드(문자열) {/*text-field-string*/}

이 예제에서 `text` 상태 변수는 문자열을 보유합니다. 입력하면 `handleChange`는 브라우저 입력 DOM 요소에서 최신 입력 값을 읽고 `setText`를 호출하여 상태를 업데이트합니다. 이렇게 하면 현재 `text`를 아래에 표시할 수 있습니다.

<Sandpack>

```js
import { useState } from 'react';

export default function MyInput() {
  const [text, setText] = useState('hello');

  function handleChange(e) {
    setText(e.target.value);
  }

  return (
    <>
      <input value={text} onChange={handleChange} />
      <p>You typed: {text}</p>
      <button onClick={() => setText('hello')}>
        Reset
      </button>
    </>
  );
}
```

</Sandpack>

<Solution />

#### 체크박스 (부울) {/*체크박스-부울*/}

이 예제에서 `좋아요` 상태 변수는 부울을 보유합니다. 입력을 클릭하면 `setLiked`는 브라우저 체크박스 입력이 선택되었는지 여부로 `liked` 상태 변수를 업데이트합니다. 좋아요` 변수는 체크박스 아래의 텍스트를 렌더링하는 데 사용됩니다.

<Sandpack>

```js
import { useState } from 'react';

export default function MyCheckbox() {
  const [liked, setLiked] = useState(true);

  function handleChange(e) {
    setLiked(e.target.checked);
  }

  return (
    <>
      <label>
        <input
          type="checkbox"
          checked={liked}
          onChange={handleChange}
        />
        I liked this
      </label>
      <p>You {liked ? 'liked' : 'did not like'} this.</p>
    </>
  );
}
```

</Sandpack>

<Solution />

#### 양식 (두 변수) {/*form-two-variables*/}

동일한 컴포넌트에 둘 이상의 상태 변수를 선언할 수 있습니다. 각 상태 변수는 완전히 독립적입니다.

<Sandpack>

```js
import { useState } from 'react';

export default function Form() {
  const [name, setName] = useState('Taylor');
  const [age, setAge] = useState(42);

  return (
    <>
      <input
        value={name}
        onChange={e => setName(e.target.value)}
      />
      <button onClick={() => setAge(age + 1)}>
        Increment age
      </button>
      <p>Hello, {name}. You are {age}.</p>
    </>
  );
}
```

```css
button { display: block; margin-top: 10px; }
```

</Sandpack>

<Solution />

</Recipes>

---

### 이전 상태를 기반으로 상태 업데이트 {/*updating-state-based-on-the-previous-state*/}

나이`가 `42`라고 가정합니다. 이 핸들러는 `setAge(age + 1)`를 세 번 호출합니다:

```js
function handleClick() {
  setAge(age + 1); // setAge(42 + 1)
  setAge(age + 1); // setAge(42 + 1)
  setAge(age + 1); // setAge(42 + 1)
}
```

그러나 한 번 클릭하면 `나이`가 `45`가 아닌 `43`이 됩니다! 이는 `set` 함수를 호출하면 이미 실행 중인 코드에서 `age` 상태 변수가 업데이트되지 않기 때문입니다. 따라서 각 `setAge(age + 1)` 호출은 `setAge(43)`이 됩니다.

이 문제를 해결하려면 다음 상태 대신 `setAge`에 *업데이터 함수***를 전달하면 됩니다:

```js [[1, 2, "a", 0], [2, 2, "a + 1"], [1, 3, "a", 0], [2, 3, "a + 1"], [1, 4, "a", 0], [2, 4, "a + 1"]]
function handleClick() {
  setAge(a => a + 1); // setAge(42 => 43)
  setAge(a => a + 1); // setAge(43 => 44)
  setAge(a => a + 1); // setAge(44 => 45)
}
```

여기서 `a => a + 1`은 업데이터 함수입니다. 이 함수는 <CodeStep step={1}>보류 상태</CodeStep>를 가져와서 그로부터 <CodeStep step={2}>다음 상태</CodeStep>를 계산합니다.

React는 업데이터 함수를 [queue.](/learn/queueing-a-series-of-state-updates)에 저장한 다음, 다음 렌더링 중에 같은 순서로 함수를 호출합니다:

1. a => a + 1`은 보류 중인 상태로 `42`를 수신하고 다음 상태로 `43`을 반환합니다.
1. `a => a + 1`은 `43`을 보류 상태로 수신하고 다음 상태로 `44`를 반환합니다.
1. a => a + 1`은 `44`를 보류 상태로 수신하고 다음 상태로 `45`를 반환합니다.

대기 중인 다른 업데이트가 없으므로 React는 결국 `45`를 현재 상태로 저장합니다.

일반적으로 보류 중인 상태 인수의 이름은 상태 변수 이름의 첫 글자, 예를 들어 `age`의 경우 `a`로 지정하는 것이 일반적입니다. 그러나 `prevAge` 또는 더 명확하다고 생각되는 다른 이름으로 부를 수도 있습니다.

React는 개발 과정에서 [순수한지 확인하기 위해](/learn/keeping-components-pure) 업데이터를 두 번 호출할 수 있습니다(#my-initializer-or-updater-function-runs-twice).


<DeepDive>

#### 항상 업데이터 함수를 사용하는 것이 좋나요? {/*is-using-an-updater-always-preferred*/}

설정하려는 상태가 이전 상태에서 계산되는 경우 항상 `setAge(a => a + 1)`와 같은 코드를 작성하라는 권고를 들을 수 있습니다. 나쁠 것은 없지만 항상 필요한 것은 아닙니다.

대부분의 경우 이 두 가지 접근법 사이에는 차이가 없습니다. React는 클릭과 같은 의도적인 사용자 행동에 대해 항상 다음 클릭 전에 `age` 상태 변수가 업데이트되도록 합니다. 즉, 클릭 핸들러가 이벤트 핸들러의 시작 부분에 "오래된" `age`를 볼 위험이 없습니다.

그러나 동일한 이벤트 내에서 여러 번 업데이트를 수행하는 경우 업데이터 함수가 유용할 수 있습니다. 상태 변수 자체에 액세스하는 것이 불편한 경우에도 유용합니다(리렌더를 최적화할 때 이 문제가 발생할 수 있습니다).

조금 더 자세한 구문보다 일관성을 선호하는 경우 설정하려는 상태가 이전 상태에서 계산되는 경우 항상 업데이터를 작성하는 것이 합리적입니다. 다른 상태 변수의 이전 상태에서 계산되는 경우, 이를 하나의 객체로 결합하고 [리듀서를 사용하는 것이 좋습니다.](/learn/extracting-state-logic-into-a-reducer)

</DeepDive>

<Recipes titleText="업데이터 함수를 전달하는 것과 다음 state를 직접 전달하는 것의 차이점" titleId="examples-updater">

#### 업데이터 함수 전달 {/*passing-the-updater-function*/}

이 예제는 업데이터 함수를 전달하므로 "+3" 버튼이 작동합니다.

<Sandpack>

```js
import { useState } from 'react';

export default function Counter() {
  const [age, setAge] = useState(42);

  function increment() {
    setAge(a => a + 1);
  }

  return (
    <>
      <h1>Your age: {age}</h1>
      <button onClick={() => {
        increment();
        increment();
        increment();
      }}>+3</button>
      <button onClick={() => {
        increment();
      }}>+1</button>
    </>
  );
}
```

```css
button { display: block; margin: 10px; font-size: 20px; }
h1 { display: block; margin: 10px; }
```

</Sandpack>

<Solution />

#### 다음 상태를 직접 전달 {/*-passing-the-next-state-directly*/}

이 예제에서는 업데이터 함수를 전달하지 않으므로 "+3" 버튼이 **의도한 대로 작동하지 않습니다**.

<Sandpack>

```js
import { useState } from 'react';

export default function Counter() {
  const [age, setAge] = useState(42);

  function increment() {
    setAge(age + 1);
  }

  return (
    <>
      <h1>Your age: {age}</h1>
      <button onClick={() => {
        increment();
        increment();
        increment();
      }}>+3</button>
      <button onClick={() => {
        increment();
      }}>+1</button>
    </>
  );
}
```

```css
button { display: block; margin: 10px; font-size: 20px; }
h1 { display: block; margin: 10px; }
```

</Sandpack>

<Solution />

</Recipes>

---

### 상태의 객체 및 배열 업데이트하기 {/*updating-objects-and-arrays-in-state*/}

객체와 배열을 state에 넣을 수 있습니다. React에서 state는 읽기 전용으로 간주되므로 **기존 객체를 *변경*하는 것이 아니라 **대체*해야 합니다**. 예를 들어, state에 'form' 객체가 있다면 변경하지 마세요:


```js
// 🚩 Don't mutate an object in state like this:
form.firstName = 'Taylor';
```

Instead, replace the whole object by creating a new one:

```js
// ✅ Replace state with a new object
setForm({
  ...form,
  firstName: 'Taylor'
});
```

자세한 내용은 [state 오브젝트 업데이트하기](/learn/updating-objects-in-state) 및 [state 배열 업데이트하기](/learn/updating-arrays-in-state) 문서를 참조하세요.

<Recipes titleText="Examples of objects and arrays in state" titleId="examples-objects">

#### Form (object) {/*form-object*/}

이 예제에서 `form` 상태 변수는 객체를 보유합니다. 각 입력에는 전체 양식의 다음 상태로 `setForm`을 호출하는 변경 핸들러가 있습니다. 스프레드 구문 `{ ...form }`은 상태 객체가 변경되지 않고 대체되도록 합니다.

<Sandpack>

```js
import { useState } from 'react';

export default function Form() {
  const [form, setForm] = useState({
    firstName: 'Barbara',
    lastName: 'Hepworth',
    email: 'bhepworth@sculpture.com',
  });

  return (
    <>
      <label>
        First name:
        <input
          value={form.firstName}
          onChange={e => {
            setForm({
              ...form,
              firstName: e.target.value
            });
          }}
        />
      </label>
      <label>
        Last name:
        <input
          value={form.lastName}
          onChange={e => {
            setForm({
              ...form,
              lastName: e.target.value
            });
          }}
        />
      </label>
      <label>
        Email:
        <input
          value={form.email}
          onChange={e => {
            setForm({
              ...form,
              email: e.target.value
            });
          }}
        />
      </label>
      <p>
        {form.firstName}{' '}
        {form.lastName}{' '}
        ({form.email})
      </p>
    </>
  );
}
```

```css
label { display: block; }
input { margin-left: 5px; }
```

</Sandpack>

<Solution />

#### 폼 (중첩된 오브젝트) {/*form-nested-object*/}

이 예제에서는 상태가 더 중첩되어 있습니다. 중첩된 상태를 업데이트할 때는 업데이트하려는 객체의 복사본과 그 위에 있는 객체를 "포함하는" 객체를 모두 만들어야 합니다. 자세한 내용은 [중첩된 오브젝트 업데이트 하기](/learn/updating-objects-in-state#updating-a-nested-object)를 읽어 보시기 바랍니다.

<Sandpack>

```js
import { useState } from 'react';

export default function Form() {
  const [person, setPerson] = useState({
    name: 'Niki de Saint Phalle',
    artwork: {
      title: 'Blue Nana',
      city: 'Hamburg',
      image: 'https://i.imgur.com/Sd1AgUOm.jpg',
    }
  });

  function handleNameChange(e) {
    setPerson({
      ...person,
      name: e.target.value
    });
  }

  function handleTitleChange(e) {
    setPerson({
      ...person,
      artwork: {
        ...person.artwork,
        title: e.target.value
      }
    });
  }

  function handleCityChange(e) {
    setPerson({
      ...person,
      artwork: {
        ...person.artwork,
        city: e.target.value
      }
    });
  }

  function handleImageChange(e) {
    setPerson({
      ...person,
      artwork: {
        ...person.artwork,
        image: e.target.value
      }
    });
  }

  return (
    <>
      <label>
        Name:
        <input
          value={person.name}
          onChange={handleNameChange}
        />
      </label>
      <label>
        Title:
        <input
          value={person.artwork.title}
          onChange={handleTitleChange}
        />
      </label>
      <label>
        City:
        <input
          value={person.artwork.city}
          onChange={handleCityChange}
        />
      </label>
      <label>
        Image:
        <input
          value={person.artwork.image}
          onChange={handleImageChange}
        />
      </label>
      <p>
        <i>{person.artwork.title}</i>
        {' by '}
        {person.name}
        <br />
        (located in {person.artwork.city})
      </p>
      <img 
        src={person.artwork.image} 
        alt={person.artwork.title}
      />
    </>
  );
}
```

```css
label { display: block; }
input { margin-left: 5px; margin-bottom: 5px; }
img { width: 200px; height: 200px; }
```

</Sandpack>

<Solution />

#### 리스트 (배열) {/*list-array*/}

이 예제에서 `todos` 상태 변수는 배열을 가리킵니다. 각 버튼 핸들러는 해당 배열의 다음 버전으로 `setTodos`를 호출합니다. '[...todos]` 스프레드 구문, `todos.map()` 및 `todos.filter()`는 state 배열이 내부적으로 변경되기보단 교체되도록 합니다.

<Sandpack>

```js App.js
import { useState } from 'react';
import AddTodo from './AddTodo.js';
import TaskList from './TaskList.js';

let nextId = 3;
const initialTodos = [
  { id: 0, title: 'Buy milk', done: true },
  { id: 1, title: 'Eat tacos', done: false },
  { id: 2, title: 'Brew tea', done: false },
];

export default function TaskApp() {
  const [todos, setTodos] = useState(initialTodos);

  function handleAddTodo(title) {
    setTodos([
      ...todos,
      {
        id: nextId++,
        title: title,
        done: false
      }
    ]);
  }

  function handleChangeTodo(nextTodo) {
    setTodos(todos.map(t => {
      if (t.id === nextTodo.id) {
        return nextTodo;
      } else {
        return t;
      }
    }));
  }

  function handleDeleteTodo(todoId) {
    setTodos(
      todos.filter(t => t.id !== todoId)
    );
  }

  return (
    <>
      <AddTodo
        onAddTodo={handleAddTodo}
      />
      <TaskList
        todos={todos}
        onChangeTodo={handleChangeTodo}
        onDeleteTodo={handleDeleteTodo}
      />
    </>
  );
}
```

```js AddTodo.js
import { useState } from 'react';

export default function AddTodo({ onAddTodo }) {
  const [title, setTitle] = useState('');
  return (
    <>
      <input
        placeholder="Add todo"
        value={title}
        onChange={e => setTitle(e.target.value)}
      />
      <button onClick={() => {
        setTitle('');
        onAddTodo(title);
      }}>Add</button>
    </>
  )
}
```

```js TaskList.js
import { useState } from 'react';

export default function TaskList({
  todos,
  onChangeTodo,
  onDeleteTodo
}) {
  return (
    <ul>
      {todos.map(todo => (
        <li key={todo.id}>
          <Task
            todo={todo}
            onChange={onChangeTodo}
            onDelete={onDeleteTodo}
          />
        </li>
      ))}
    </ul>
  );
}

function Task({ todo, onChange, onDelete }) {
  const [isEditing, setIsEditing] = useState(false);
  let todoContent;
  if (isEditing) {
    todoContent = (
      <>
        <input
          value={todo.title}
          onChange={e => {
            onChange({
              ...todo,
              title: e.target.value
            });
          }} />
        <button onClick={() => setIsEditing(false)}>
          Save
        </button>
      </>
    );
  } else {
    todoContent = (
      <>
        {todo.title}
        <button onClick={() => setIsEditing(true)}>
          Edit
        </button>
      </>
    );
  }
  return (
    <label>
      <input
        type="checkbox"
        checked={todo.done}
        onChange={e => {
          onChange({
            ...todo,
            done: e.target.checked
          });
        }}
      />
      {todoContent}
      <button onClick={() => onDelete(todo.id)}>
        Delete
      </button>
    </label>
  );
}
```

```css
button { margin: 5px; }
li { list-style-type: none; }
ul, li { margin: 0; padding: 0; }
```

</Sandpack>

<Solution />

#### Immer로 간결한 업데이트 로직 작성하기 {/*작성하기-간결한-업데이트-로직-with-immer*/}

변이 없이 배열과 객체를 업데이트하는 것이 지루하게 느껴진다면, [Immer](https://github.com/immerjs/use-immer)와 같은 라이브러리를 사용하여 반복되는 코드를 줄일 수 있습니다. Immer를 사용하면 객체를 변경하는 것처럼 간결한 코드를 작성할 수 있지만, 내부적으로는 변경 불가능한 업데이트를 수행합니다:

<Sandpack>

```js
import { useState } from 'react';
import { useImmer } from 'use-immer';

let nextId = 3;
const initialList = [
  { id: 0, title: 'Big Bellies', seen: false },
  { id: 1, title: 'Lunar Landscape', seen: false },
  { id: 2, title: 'Terracotta Army', seen: true },
];

export default function BucketList() {
  const [list, updateList] = useImmer(initialList);

  function handleToggle(artworkId, nextSeen) {
    updateList(draft => {
      const artwork = draft.find(a =>
        a.id === artworkId
      );
      artwork.seen = nextSeen;
    });
  }

  return (
    <>
      <h1>Art Bucket List</h1>
      <h2>My list of art to see:</h2>
      <ItemList
        artworks={list}
        onToggle={handleToggle} />
    </>
  );
}

function ItemList({ artworks, onToggle }) {
  return (
    <ul>
      {artworks.map(artwork => (
        <li key={artwork.id}>
          <label>
            <input
              type="checkbox"
              checked={artwork.seen}
              onChange={e => {
                onToggle(
                  artwork.id,
                  e.target.checked
                );
              }}
            />
            {artwork.title}
          </label>
        </li>
      ))}
    </ul>
  );
}
```

```json package.json
{
  "dependencies": {
    "immer": "1.7.3",
    "react": "latest",
    "react-dom": "latest",
    "react-scripts": "latest",
    "use-immer": "0.5.1"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }
}
```

</Sandpack>

<Solution />

</Recipes>

---

### 초기 상태 재생성 방지 {/*avoiding-recreating-the-initial-state*/}

React는 초기 state를 한 번 저장하고 다음 렌더링에서 무시합니다.

```js
function TodoList() {
  const [todos, setTodos] = useState(createInitialTodos());
  // ...
```

createInitialTodos()`의 결과는 초기 렌더링에만 사용되지만 여전히 모든 렌더링에서 이 함수를 호출하게 됩니다. 이는 큰 배열을 생성하거나 값비싼 계산을 수행하는 경우 낭비가 될 수 있습니다.

이 문제를 해결하려면 이 함수를 `useState`에 _initializer_ 함수로 전달하면 됩니다:

```js
function TodoList() {
  const [todos, setTodos] = useState(createInitialTodos);
  // ...
```

함수를 호출한 결과인 `createInitialTodos()`가 아니라 *함수 자체인 `createInitialTodos`를 전달하고 있다는 것을 주목하세요. 만약 `useState`에 함수를 전달하면 React는 초기화 중에만 함수를 호출합니다.

React는 개발 중에 이니셜라이저가 [순수]한지 확인하기 위해 [이니셜라이저를 두 번 호출](#my-initializer-or-updater-function-runs-twice)할 수 있습니다.

<Recipes titleText="이니셜라이저를 전달하는 것과 초기 상태를 직접 전달하는 것의 차이점" titleId="examples-initializer">

#### 이니셜라이저 함수 전달 {/*passing-the-initializer-function*/}

이 예제에서는 이니셜라이저 함수를 전달하므로 `createInitialTodos` 함수는 초기화 중에만 실행됩니다. 사용자가 입력을 입력할 때와 같이 컴포넌트가 다시 렌더링할 때는 실행되지 않습니다.

<Sandpack>

```js
import { useState } from 'react';

function createInitialTodos() {
  const initialTodos = [];
  for (let i = 0; i < 50; i++) {
    initialTodos.push({
      id: i,
      text: 'Item ' + (i + 1)
    });
  }
  return initialTodos;
}

export default function TodoList() {
  const [todos, setTodos] = useState(createInitialTodos);
  const [text, setText] = useState('');

  return (
    <>
      <input
        value={text}
        onChange={e => setText(e.target.value)}
      />
      <button onClick={() => {
        setText('');
        setTodos([{
          id: todos.length,
          text: text
        }, ...todos]);
      }}>Add</button>
      <ul>
        {todos.map(item => (
          <li key={item.id}>
            {item.text}
          </li>
        ))}
      </ul>
    </>
  );
}
```

</Sandpack>

<Solution />

#### 초기 상태 직접 전달 {/*passing-the-initial-state-directly*/}

이 예제에서는 이니셜라이저 함수를 전달하지 않으므로, 입력을 입력할 때와 같이 모든 렌더링에서 `createInitialTodos` 함수가 실행됩니다. 동작에 관찰할 수 있는 차이는 없지만 이 코드는 효율성이 떨어집니다.

<Sandpack>

```js
import { useState } from 'react';

function createInitialTodos() {
  const initialTodos = [];
  for (let i = 0; i < 50; i++) {
    initialTodos.push({
      id: i,
      text: 'Item ' + (i + 1)
    });
  }
  return initialTodos;
}

export default function TodoList() {
  const [todos, setTodos] = useState(createInitialTodos());
  const [text, setText] = useState('');

  return (
    <>
      <input
        value={text}
        onChange={e => setText(e.target.value)}
      />
      <button onClick={() => {
        setText('');
        setTodos([{
          id: todos.length,
          text: text
        }, ...todos]);
      }}>Add</button>
      <ul>
        {todos.map(item => (
          <li key={item.id}>
            {item.text}
          </li>
        ))}
      </ul>
    </>
  );
}
```

</Sandpack>

<Solution />

</Recipes>

---

### key로 state 를 다시 설정하기 {/*resetting-state-with-a-key*/}

리스트를 렌더링할 때 `key` 속성을 자주 접하게 되지만(/learn/rendering-lists), 다른 용도로도 사용할 수 있습니다.

컴포넌트에 다른 `key`를 전달하여 컴포넌트의 상태를 **재설정할 수 있습니다.** 이 예제에서 Reset 버튼은 `버전` 상태 변수를 변경하고, `key`로 전달한 값을 `Form`에 전달합니다. 키`가 변경되면 React는 `Form` 컴포넌트(및 그 모든 자식)를 처음부터 다시 생성하므로 상태가 초기화됩니다.

자세한 내용은 [state 보존 및 초기화](/learn/preserving-and-resetting-state)를 읽어보세요.

<Sandpack>

```js App.js
import { useState } from 'react';

export default function App() {
  const [version, setVersion] = useState(0);

  function handleReset() {
    setVersion(version + 1);
  }

  return (
    <>
      <button onClick={handleReset}>Reset</button>
      <Form key={version} />
    </>
  );
}

function Form() {
  const [name, setName] = useState('Taylor');

  return (
    <>
      <input
        value={name}
        onChange={e => setName(e.target.value)}
      />
      <p>Hello, {name}.</p>
    </>
  );
}
```

```css
button { display: block; margin-bottom: 20px; }
```

</Sandpack>

---

### 이전 렌더링의 정보 저장 {/*storing-information-from-previous-renders*/}

일반적으로 이벤트 핸들러에서 상태를 업데이트합니다. 그러나 드물게 렌더링에 대한 응답으로 상태를 조정하고 싶을 수도 있습니다(예: 소품이 변경될 때 상태 변수를 변경하고 싶을 수 있습니다).

대부분의 경우 이 기능은 필요하지 않습니다:

* 필요한 값을 현재 프롭이나 다른 상태로부터 완전히 계산할 수 있다면, [해당 중복 상태를 모두 제거하세요.](/learn/choosing-the-state-structure#avoid-redundant-state)** 너무 자주 재계산하는 것이 걱정된다면, [`useMemo` Hook](/reference/react/useMemo)이 도움이 될 수 있습니다.
* 전체 컴포넌트 트리의 상태를 초기화하려면 [컴포넌트에 다른 `키`를 전달하세요.](#resetting-state-with-a-key).
* 가능하면 이벤트 핸들러에서 모든 관련 상태를 업데이트하세요.

이 중 어느 것도 적용되지 않는 드문 경우, 컴포넌트가 렌더링되는 동안 `set` 함수를 호출하여 지금까지 렌더링된 값을 기반으로 상태를 업데이트하는 데 사용할 수 있는 패턴이 있습니다.

다음은 예시입니다. 이 `CountLabel` 컴포넌트는 전달된 `count` 프로퍼티를 표시합니다:

```js CountLabel.js
export default function CountLabel({ count }) {
  return <h1>{count}</h1>
}
```

마지막 변경 이후 카운터가 *증가 또는 감소*했는지 표시하고 싶다고 가정해 보겠습니다. `count` 프로퍼티는 이를 알려주지 않으므로 이전 값을 추적해야 합니다. 이를 추적하기 위해 `prevCount` 상태 변수를 추가합니다. `trend`라는 다른 상태 변수를 추가하여 count의 증가 또는 감소 여부를 유지합니다. `prevCount`와 `count`를 비교하고, 같지 않으면 `prevCount`와 `trend`를 모두 업데이트합니다. 이제 현재의 count prop과 *마지막 렌더링 이후 어떻게 변화했는지*를 모두 표시할 수 있습니다.
<Sandpack>

```js App.js
import { useState } from 'react';
import CountLabel from './CountLabel.js';

export default function App() {
  const [count, setCount] = useState(0);
  return (
    <>
      <button onClick={() => setCount(count + 1)}>
        Increment
      </button>
      <button onClick={() => setCount(count - 1)}>
        Decrement
      </button>
      <CountLabel count={count} />
    </>
  );
}
```

```js CountLabel.js active
import { useState } from 'react';

export default function CountLabel({ count }) {
  const [prevCount, setPrevCount] = useState(count);
  const [trend, setTrend] = useState(null);
  if (prevCount !== count) {
    setPrevCount(count);
    setTrend(count > prevCount ? 'increasing' : 'decreasing');
  }
  return (
    <>
      <h1>{count}</h1>
      {trend && <p>The count is {trend}</p>}
    </>
  );
}
```

```css
button { margin-bottom: 10px; }
```

</Sandpack>

렌더링하는 동안 `set` 함수를 호출하는 경우 `prevCount !== count`와 같은 조건 안에 있어야 하며, 조건 안에 `setPrevCount(count)`와 같은 호출이 있어야 한다는 점에 유의하세요. 그렇지 않으면 컴포넌트가 충돌할 때까지 루프에서 다시 렌더링됩니다. 또한 다음과 같이 *현재 렌더링 중인* 컴포넌트의 상태만 업데이트할 수 있습니다. 렌더링 중에 *다른* 컴포넌트의 `set` 함수를 호출하는 것은 오류입니다. 마지막으로, `set` 호출은 여전히 [변이 없이 상태 업데이트](#updating-objects-and-arrays-in-state)를 해야 하며, 이는 [순수 함수]의 다른 규칙을 위반할 수 없다는 것을 의미하지는 않습니다.

이 패턴은 이해하기 어려울 수 있으며 일반적으로 피하는 것이 가장 좋습니다. 하지만 이펙트에서 상태를 업데이트하는 것보다 낫습니다. 렌더링 도중 `set` 함수를 호출하면 React는 컴포넌트가 `return` 문으로 종료된 직후와 자식들을 렌더링하기 전에 해당 컴포넌트를 다시 렌더링합니다. 이렇게 하면 자식 컴포넌트를 두 번 렌더링할 필요가 없습니다. 나머지 컴포넌트 함수는 계속 실행되고 결과는 버려집니다. 조건이 모든 Hook 호출보다 낮은 경우, `return;`을 추가하여 렌더링을 더 일찍 다시 시작할 수 있습니다.

---

## 문제 해결 {/*troubleshooting*/}

### 상태를 업데이트했지만 로깅하면 이전 값이 나옵니다 {/*ive-updated-the-state-but-logging-gives-me-the-old-value*/}

'set' 함수를 호출해도 **실행 중인 코드에서 상태가 변경되지 않습니다**:


```js {4,5,8}
function handleClick() {
  console.log(count);  // 0

  setCount(count + 1); // Request a re-render with 1
  console.log(count);  // Still 0!

  setTimeout(() => {
    console.log(count); // Also 0!
  }, 5000);
}
```

상태는 스냅샷처럼 동작하기 때문입니다. (/learn/state-as-a-snapshot) 상태를 업데이트하면 새 상태 값으로 다른 렌더링을 요청하지만 이미 실행 중인 이벤트 핸들러의 `count` JavaScript 변수에는 영향을 미치지 않습니다.

다음 상태를 사용해야 하는 경우 `set` 함수에 전달하기 전에 변수에 저장할 수 있습니다:

```js
const nextCount = count + 1;
setCount(nextCount);

console.log(count);     // 0
console.log(nextCount); // 1
```

---

### state를 업데이트했지만 화면이 업데이트되지 않습니다 {/*ive-updated-the-state-but-the-screen-doesnt-update*/}

React는 [`Object.is`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is) 비교에 의해 결정된 대로 다음 상태가 이전 상태와 같으면 업데이트를 **무시합니다. 이는 보통 객체나 배열의 상태를 직접 변경할 때 발생합니다:
```js
obj.x = 10;  // 🚩 Wrong: mutating existing object
setObj(obj); // 🚩 Doesn't do anything
```

기존 `obj` 객체를 돌연변이시킨 후 다시 `setObj`로 전달했기 때문에 React가 업데이트를 무시했습니다. 이 문제를 해결하려면 객체와 배열을 _변이시키는 대신 항상 상태의 객체와 배열을 _교체_(#updating-objects-and-arrays-in-state)하도록 해야 합니다:
```js
// ✅ Correct: creating a new object
setObj({
  ...obj,
  x: 10
});
```

---

### 오류가 발생했습니다: "너무 많은 리렌더링" {/*im-getting-an-error-too-many-re-renders*/}

`Too many re-renders. React limits the number of renders to prevent an infinite loop.` ('너무 많은 리렌더링, React는 무한 루프를 방지하기 위해 렌더링 횟수를 제한합니다.') 와 같은 오류가 발생할 수 있습니다. 일반적으로 이것은 *렌더링 중* 상태를 무조건적으로 설정해 무한루프 상태에 들어가고 있음을 의미합니다: 렌더링, state 설정(렌더링 유발), 렌더링, state 설정(렌더링을 유발), ... 이 문제는 이벤트 핸들러를 지정하다가 실수로 인해 발생하곤 합니다:

```js {1-2}
// 🚩 Wrong: calls the handler during render
return <button onClick={handleClick()}>Click me</button>

// ✅ Correct: passes down the event handler
return <button onClick={handleClick}>Click me</button>

// ✅ Correct: passes down an inline function
return <button onClick={(e) => handleClick(e)}>Click me</button>
```

이 오류의 원인을 찾을 수 없는 경우 콘솔에서 오류 옆에 있는 화살표를 클릭하고 자바스크립트 스택을 살펴보고 오류의 원인이 되는 특정 `set` 함수 호출을 찾아보세요.

---

### 내 초기화 또는 업데이터 함수가 두 번 실행됨 {/*my-initializer-or-updater-function-runs-twice*/}

[Strict 모드](/reference/react/StrictMode)에서는, React가 일부 함수를 한 번이 아닌 두 번 호출합니다.

```js {2,5-6,11-12}
function TodoList() {
  // This component function will run twice for every render.

  const [todos, setTodos] = useState(() => {
    // This initializer function will run twice during initialization.
    return createTodos();
  });

  function handleClick() {
    setTodos(prevTodos => {
      // This updater function will run twice for every click.
      return [...prevTodos, createTodo()];
    });
  }
  // ...
```

이는 의도된 동작이며 이렇게 작동함으로서 문제가 발생되지 않아야 합니다.

이 **개발 전용** 동작은 컴포넌트를 [순수하게 유지](/learn/keeping-components-pure)하는 데 도움이 됩니다. React는 호출 중 하나의 결과를 사용하고 다른 호출의 결과는 무시합니다. 컴포넌트, 이니셜라이저, 업데이터 함수가 순수하다면 로직에 영향을 미치지 않을 것입니다. 그러나 실수로 impure한 경우, 이 동작은 실수를 알아차리는 데 도움이 됩니다.

예를 들어, 이 impure한 업데이터 함수는 상태의 배열을 변경합니다:

```js {2,3}
setTodos(prevTodos => {
  // 🚩 Mistake: mutating state
  prevTodos.push(createTodo());
});
```

React는 업데이터 함수를 두 번 호출하기 때문에 할 일이 두 번 추가되었음을 알 수 있으므로 실수가 있다는 것을 알 수 있습니다. 이 예제에서는 [배열을 변경하는 대신 배열을 교체](#updating-objects-and-arrays-in-state)하여 실수를 수정할 수 있습니다:

```js {2,3}
setTodos(prevTodos => {
  // ✅ Correct: replacing with new state
  return [...prevTodos, createTodo()];
});
```

이제 이 업데이터 함수는 순수 함수이므로 한 번 더 호출해도 동작에 차이가 없습니다. 그렇기 때문에 React에서 두 번 호출하면 실수를 찾는 데 도움이 됩니다. **컴포넌트, 이니셜라이저, 업데이터 함수만 순수해야 합니다.** 이벤트 핸들러는 순수할 필요가 없으므로 React는 절대 이벤트 핸들러를 두 번 호출하지 않습니다.

자세한 내용은 [컴포넌트 순수하게 유지하기](/learn/keeping-components-pure)를 읽어보세요.

---

### 상태를 함수로 설정하려고 하는데 대신 함수가 호출됩니다 {/*im-trying-to-set-state-to-a-function-but-it-gets-called-instead*/}. {/*상태를-함수로-설정하려고-하는데-대신-함수가-호출됩니다-im-trying-to-set-state-to-a-function-but-it-gets-called-instead*/}

이런 식으로 함수를 state에 넣을 수 없습니다:

```js
const [fn, setFn] = useState(someFunction);

function handleClick() {
  setFn(someOtherFunction);
}
```

함수를 전달하기 때문에 React는 `someFunction`이 [초기화 함수](#avoiding-recreating-the-initial-state)이고, `someOtherFunction`이 [업데이터 함수](#updater-state-based-on-the-previous-state)라고 가정하여 이를 호출하고 결과를 저장하려고 시도합니다. 실제로 함수를 저장하려면 두 경우 모두 함수 앞에 '() =>`를 넣어야 합니다. 그러면 React가 전달한 함수를 저장합니다.

```js {1,4}
const [fn, setFn] = useState(() => someFunction);

function handleClick() {
  setFn(() => someOtherFunction);
}
```
