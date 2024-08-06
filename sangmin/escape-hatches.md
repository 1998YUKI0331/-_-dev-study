# [탈출구](https://ko.react.dev/learn/escape-hatches)

## Ref로 값 참조하기

### > ref와 state의 차이

| refs | state |
| -------------------------------- | ---------------------------------- |
| useRef(initialValue)는 {current: initialValue}를 반환 | useState(initialValue)는 state 변수의 현재 값과 setter 함수 [value, setValue] 반환 |
| state를 바꿔도 리렌더 되지 않음 | state를 바꾸면 리렌더 |
| Mutable-렌더링 프로세스 외부에서 current 값을 수정 및 업데이트 가능 | ”Immutable”—state 를 수정하기 위해서는 state 설정 함수를 사용하여 리렌더 대기열에 삽입 |
| 렌더링 중에는 current 값을 읽거나 쓰기 금지 | 언제든지 state를 읽을 수 있지만, 각 렌더마다 변경되지 않는 자체적인 state의 snapshot이 존재 |

#### Q. '렌더링 중'이란 어떤 시점을 말하는 것일까?

#### A. "렌더링 중"이라는 표현은 React 컴포넌트의 렌더링 단계 동안의 특정 시점을 의미한다. <br/> 이 시점은 컴포넌트가 새로운 상태나 props를 반영하기 위해 DOM 업데이트를 준비하는 과정. <br/>React에서 갱신(렌더링)은 두 단계로 나눌 수 있다.

**렌더 단계**
- 컴포넌트가 함수로 호출되고 JSX를 반환하는 단계. 
- React는 컴포넌트의 새로운 상태와 props를 기반으로 가상 DOM을 생성. (공식 문서에서는 사용하지 않는 표현)
- 부수 효과가 없는 순수 함수처럼 동작.
- current 값을 읽거나 쓰지 말아야 하는 시점.

**커밋 단계**
- React가 실제 DOM을 업데이트하는 단계.
- DOM 업데이트와 함께 부수 효과를 처리.
- useEffect, useLayoutEffect, componentDidMount, componentDidUpdate와 같은 라이프사이클 메서드가 호출.
- current 값을 읽거나 쓸 수 있는 시점.

"렌더링 중"이란, 컴포넌트가 함수로 호출되어 JSX를 반환하는 **렌더 단계(Render Phase)**를 의미.

이 과정에서 React는 순수 함수처럼 컴포넌트를 실행하기 때문에, 부수 효과(side effect)가 발생하지 않도록 해야 한다. <br/> 따라서, ref의 current 값을 읽거나 쓰는 작업도 피해야 하는 것.

다음은 useRef를 잘못 사용하는 에제.

```typescript
function MyComponent() {
  const myRef = useRef(0);

  // 렌더 단계에서 current 값을 변경하는 잘못된 사용
  myRef.current = myRef.current + 1;

  return <div>Current: {myRef.current}</div>;
}
```

<br/>

### > useRef의 내부적 동작

useState와 useRef가 모두 React에 의해 제공되지만, 원칙적으로 useRef는 useState 위에 구현될 수 있다. <br/>
React 내부에서 useRef가 아래와같은 형태로 구현되는 것을 상상할 수 있다.

```typescript
function useRef(initialValue) {
  const [ref, unused] = useState({ current: initialValue });
  return ref;
}
```

첫 번째 렌더 중에 useRef는 { current: initialValue }을 반환. <br/>
이 객체는 React에 의해 저장되므로 다음 렌더 중에 같은 객체가 반환. <br/>
useRef는 항상 동일한 객체를 반환해야 하므로 state setter함수는 필요하지 않다.

React는 useRef가 실제로 충분히 일반적이기 때문에 built-in 버전을 제공. <br/>
setter가 없는 일반적인 state 변수라고 생각할 수 있다. 객체 지향 프로그래밍에 익숙하다면 refs는 인스턴스 필드를 상기시킬 수 있다<br/>
하지만 this.something 대신에 somethingRef.current 처럼 써야한다.


<br/>

### > refs의 사용 예시

refs를 *escape hatch로 간주한다. 

애플리케이션 로직과 데이터 흐름의 상당 부분이 refs에 의존한다면 접근 방식을 재고해 보는 것이 좋다. <br/>
ref.current가 언제 변하는지 React는 모르기 때문에 렌더링할 때 읽어도 컴포넌트의 동작을 예측하기 어렵다. 

React state의 제한은 refs에 적용되지 않는다. <br/>
예를 들어 state는 모든 렌더링에 대한 snapshot 및 동기적으로 업데이트되지 않는 것과 같이 작동한다. <br/>
그러나 ref의 current 값을 변조하면 다음과 같이 즉시 변경.

```typescript
ref.current = 5;
console.log(ref.current); // 5
```
그 이유는 ref 자체가 일반 자바스크립트 객체처럼 동작하기 때문.

ref로 작업할 때 *mutation 방지에 대해 걱정할 필요가 없다. <br/>
변형하는 객체가 렌더링에 사용되지 않는 한, React는 ref 혹은 해당 콘텐츠를 어떻게 처리하든 신경 쓰지 않는다.

*escape hatch: Escape Hatch는 일반적으로 복잡한 시스템에서 일반적인 규칙이나 제약을 우회하거나 벗어날 수 있는 방법을 의미.

*mutation 방지: Mutation 방지는 객체나 데이터 구조를 변경하지 않고, 불변성을 유지하면서 상태를 관리하는 개념. <br/>
React에서 상태를 업데이트할 때, 직접적으로 객체나 배열을 변형하지 않고, 새로운 객체나 배열을 생성하여 상태를 업데이트하는 것이 권장됨.

<br/>

### [> How can I measure a DOM node?](https://legacy.reactjs.org/docs/hooks-faq.html#how-can-i-measure-a-dom-node)

추가 참조: [useEffect와 useRef의 사용](https://velog.io/@shmoon2917/useEffect-%EC%9D%98%EC%A1%B4%EC%84%B1%EC%97%90-ref%EB%A5%BC-%EB%8B%B4%EC%9D%84-%EB%95%8C%EB%A7%88%EB%8B%A4-%EC%B0%9C%EC%B0%9C%ED%95%98%EC%8B%A0-%EB%B6%84%EB%93%A4%EC%9D%84-%EC%9C%84%ED%95%B4)

DOM 노드의 위치나 크기를 측정하는 기본적인 방법 중 하나는 콜백 ref를 사용하는 것. <br/>
React는 ref가 다른 노드에 연결될 때마다 해당 콜백을 호출.

```typescript
function MeasureExample() {
  const [height, setHeight] = useState(0);

  const measuredRef = useCallback(node => {
    if (node !== null) {
      setHeight(node.getBoundingClientRect().height);
    }
  }, []);

  return (
    <>
      <h1 ref={measuredRef}>Hello, world</h1>
      <h2>The above header is {Math.round(height)}px tall</h2>
    </>
  );
}
```
useCallback 참조를 사용하면 자식 구성 요소가 나중에 측정된 노드를 표시하더라도 (ex. 클릭에 대한 응답으로) 부모 구성 요소에서 여전히 알림을 받고 측정을 업데이트할 수 있다.

이렇게 하면 ref 콜백이 다시 렌더링하는 동안 변경되지 않으므로 React가 불필요하게 호출하지 않는다.

이 예에서 콜백 참조는 렌더링된 구성 요소가 모든 재렌더링에서 존재하기 때문에 구성 요소가 마운트되고 언마운트될 때만 호출된다. <br/>
구성 요소의 크기가 조정될 때마다 알림을 받으려면 또는 이를 기반으로 빌드된 타사 Hook을 사용할 수 있다.


<br/>
<br/>

## Ref로 DOM 조작하기

### > ref 콜백을 사용하여 ref 리스트 관리

때때로 목록의 아이템마다 ref가 필요할 수도 있고, 얼마나 많은 ref가 필요할지 예측할 수 없는 경우도 있다. <br/>
그럴 때 아래 코드는 작동하지 않는다.
 
```typescript
<ul>
  {items.map((item) => {
    const ref = useRef(null);
    return <li ref={ref} />;
  })}
</ul>
```

Hook은 컴포넌트의 최상단에서만 호출되어야 하기 때문. <br/>
useRef를 반복문, 조건문 혹은 map() 안쪽에서 호출할 수 없다.

문제를 해결하는 한 방법은 부모 요소에서 단일 ref를 얻고, querySelectorAll과 같은 DOM 조작 메서드를 사용하여 개별 자식 노드를 “찾는” 것. <br/>
하지만 이는 다루기가 힘들며 DOM 구조가 바뀌는 경우 작동하지 않을 수 있다.

또 다른 해결책은 ref 어트리뷰트에 함수를 전달하는 것. 이것을 “ref 콜백”이라고 한다. <br/>
React는 ref를 설정할 때 DOM 노드와 함께 ref 콜백을 호출하며, ref를 지울 때에는 null을 전달한다. <br/>
이를 통해 자체 배열이나 Map을 유지하고, 인덱스나 특정 ID를 사용하여 어떤 ref에든 접근할 수 있다.

아래 예시는 긴 목록에서 특정 노드에 스크롤 하기 위해 앞에서 말한 접근법을 사용한다.

<br/>

### > 커스텀 컴포넌트의 DOM 노트 접근하기

```
Warning: Function components cannot be given refs. Attempts to access this ref will fail. Did you mean to use React.forwardRef()?
```

React는 기본적으로 다른 컴포넌트의 DOM 노드에 접근하는 것을 허용하지 않는다. 이것은 의도적인 설계. <br/> 
Ref는 자제해서 사용해야 하는 탈출구. 직접 다른 컴포넌트의 DOM 노드를 조작하는 것은 코드가 쉽게 깨지게 만든다.

대신 특정 컴포넌트에서 소유한 DOM 노드를 선택적으로 노출할 수 있다. <br/>
컴포넌트는 자식 중 하나에 ref를 “전달”하도록 지정할 수 있다.

```typescript
const MyInput = forwardRef((props, ref) => {
  return <input {...props} ref={ref} />;
});
```

\<MyInput ref={inputRef} />으로 React가 대응되는 DOM 노드를 inputRef.current에 대입하도록 설정. <br/>
하지만 이것은 전적으로 MyInput 컴포넌트의 선택에 달려 있으며, 기본적으로는 이렇게 동작하지 않는다. <br/>
MyInput은 자체적으로 수신받은 ref를 컴포넌트 내부의 \<input>으로 전달한다.

<br/>

### > 명령형 처리방식으로 하위 API 노출

부모 컴포넌트에서 DOM 노드의 CSS 스타일을 직접 변경하는 등의 예상치 못 한 작업을 할 수 있다. <br/>
몇몇 상황에서는 노출된 기능을 제한하고 싶을 수 있는데, useImperativeHandle을 사용.

```typescript
import {
  forwardRef,
  useRef,
  useImperativeHandle
} from 'react';

const MyInput = forwardRef((props, ref) => {
  const realInputRef = useRef(null);
  useImperativeHandle(ref, () => ({
    // 오직 focus만 노출합니다.
    focus() {
      realInputRef.current.focus();
    },
  }));
  return <input {...props} ref={realInputRef} />;
});

export default function Form() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
  }

  return (
    <>
      <MyInput ref={inputRef} />
      <button onClick={handleClick}>
        Focus the input
      </button>
    </>
  );
}
```

여기 MyInput 내부의 realInputRef는 실제 input DOM 노드를 가지고 있다. <br/>
하지만 useImperativeHandle을 사용하여 React가 ref를 참조하는 부모 컴포넌트에 직접 구성한 객체를 전달하도록 지시한다. <br/> 
따라서 Form 컴포넌트 안쪽의 inputRef.current는 foucs 메서드만 가지고 있다. <br/>
이 경우 ref는 DOM 노드가 아니라 useImperativeHandle 호출에서 직접 구성한 객체.

<br/>

### > flushSync로 state 변경을 동적으로 플러시하기

새로운 할 일을 추가하고 할 일 목록의 마지막으로 화면 스크롤을 내리는 동작을 하는 아래 코드. <br/>
어떤 이유에 의해 마지막으로 추가된 할 일의 직전으로 항상 스크롤 되는 것을 관찰할 수 있다.

```typescript
import { useState, useRef } from 'react';

export default function TodoList() {
  const listRef = useRef(null);
  const [text, setText] = useState('');
  const [todos, setTodos] = useState(
    initialTodos
  );

  function handleAdd() {
    const newTodo = { id: nextId++, text: text };
    setText('');
    setTodos([ ...todos, newTodo]);
    listRef.current.lastChild.scrollIntoView({
      behavior: 'smooth',
      block: 'nearest'
    });
  }

  return (
    <>
      <button onClick={handleAdd}>
        Add
      </button>
      <input
        value={text}
        onChange={e => setText(e.target.value)}
      />
      <ul ref={listRef}>
        {todos.map(todo => (
          <li key={todo.id}>{todo.text}</li>
        ))}
      </ul>
    </>
  );
}

let nextId = 0;
let initialTodos = [];
for (let i = 0; i < 20; i++) {
  initialTodos.push({
    id: nextId++,
    text: 'Todo #' + (i + 1)
  });
}
```

문제는 다음 두 줄에 있습니다.

```typescript
setTodos([ ...todos, newTodo]);
listRef.current.lastChild.scrollIntoView();
```
React에서 state 갱신은 큐에 쌓여 비동기적으로 처리된다. <br/>
여기에선 setTodos가 DOM을 바로 업데이트하지 않기 때문에 문제가 발생한다. <br/>
그래서 할 일 목록의 마지막 노드로 스크롤 할 때, DOM에 아직 새로운 할 일이 추가되지 않은 상태. <br/>
위 예시에서 스크롤이 계속 한 항목에 뒤처지는 이유이다.

이 문제를 고치기 위해 React가 DOM 변경을 동기적으로 수행하도록 할 수 있다. <br/>
이를 위해 react-dom 패키지의 flushSync를 가져오고 state 업데이트를 flushSync 호출로 감싸면 된다.

```typescript
flushSync(() => {
  setTodos([ ...todos, newTodo]);
});

listRef.current.lastChild.scrollIntoView();
```
위의 내용은 flushSync로 감싼 코드가 실행된 직후 React가 동기적으로 DOM을 변경하도록 지시. <br/>
결과적으로 마지막 할 일은 스크롤 하기 전에 항상 DOM에 추가되어 있을 것.

```typescript
import { useState, useRef } from 'react';
import { flushSync } from 'react-dom';

export default function TodoList() {
  const listRef = useRef(null);
  const [text, setText] = useState('');
  const [todos, setTodos] = useState(
    initialTodos
  );

  function handleAdd() {
    const newTodo = { id: nextId++, text: text };
    flushSync(() => {
      setText('');
      setTodos([ ...todos, newTodo]);
    });
    listRef.current.lastChild.scrollIntoView({
      behavior: 'smooth',
      block: 'nearest'
    });
  }

  return (
    <>
      <button onClick={handleAdd}>
        Add
      </button>
      <input
        value={text}
        onChange={e => setText(e.target.value)}
      />
      <ul ref={listRef}>
        {todos.map(todo => (
          <li key={todo.id}>{todo.text}</li>
        ))}
      </ul>
    </>
  );
}

let nextId = 0;
let initialTodos = [];
for (let i = 0; i < 20; i++) {
  initialTodos.push({
    id: nextId++,
    text: 'Todo #' + (i + 1)
  });
}
```

<br/>

### > ref로 DOM을 조작하는 것에 대하여

Ref는 탈출구. “React에서 벗어나야 할 때”만 사용한다. <br/>
포커스 혹은 스크롤 위치를 관리하거나, React가 노출하지 않는 브라우저 API를 호출하는 등의 작업이 이에 포함된다.

포커스 및 스크롤 관리 같은 비 파괴적인 행동을 고수한다면 어떤 문제도 마주치지 않을 것. <br/>
하지만 DOM을 직접 수정하는 시도를 한다면 React가 만들어 내는 변경 사항과 충돌을 발생시킬 위험을 감수해야 한다.

<br/>
<br/>

## Effect로 동기화하기

<br/>
<br/>

## Effect가 필요하지 않은 경우

<br/>
<br/>

## React Effect의 생명주기

<br/>
<br/>

## Effect에서 이벤트 분리하기

<br/>
<br/>

## Effect의 의존성 제거하기

<br/>
<br/>

## 커스텀 Hook으로 로직 재사용하기

<br/>
<br/>