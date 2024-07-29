# [State 관리하기](https://ko.react.dev/learn/managing-state)

state가 어떻게 구성되는지, 컴포넌트 간에 어떻게 state가 흐르는지에 대해 의식적으로 파악하면 도움이 된다. 이 장에서는 **state를 잘 구성하는 방법, state 업데이트 로직을 유지 보수 가능하게 관리하는 방법, 멀리 있는 컴포넌트 간에 state를 공유하는 방법**에 대해 알아보자.

## State를 사용해 Input 다루기

리액트는 선언적인 방식으로 UI를 조작한다. 개별적인 UI를 조작하는 것이 아닌 컴포넌트 내부에 여러 state를 묘사하고 사용자의 입력에 따라 state를 변경한다.

### 선언형 UI와 명령형 UI 비교

명령형 UI는 상호작용을 위해 발생할 상황에 따른 지침을 작성해야 한다. 반면에 **선언형 UI는 무엇을 보여주고 싶은지 선언**하기만 하면 된다.

- 컴포넌트의 다양한 시각적 state를 확인
- 무엇이 state 변화를 트리거 하는지 추적
- `useState`를 사용해서 메모리의 state 표현
- 불필요한 state 제거
- state 설정을 위해 이벤트 핸들러 연결
  > [Coding Styles: Imperative, Declarative and DSL 🤯](https://www.linkedin.com/pulse/coding-styles-imperative-declarative-dsl-sameer-kumar)

### 123: state 확인하고, 트리거 추적하고 표현하기

- 사용자가 볼 수 있는 UI의 모든 state를 시각화 한다.
- 그리고 두 가지 인풋 유형(휴먼 인풋, 컴퓨터 인풋)에 UI를 업데이트 하기 위해 state 변수를 설정한다.
- 다음으로, 다음으로 `useState`를 사용하여 컴포넌트의 state를 최대한 **단순하고 간결하게** 표현한다.

### 4: 불필요한 state 변수를 제거하기

state는 단순할 수록 좋다고 했다. 따라서 중복은 피하고 필수적인 state만 남겨야 한다. 리팩토링의 목표는 **state가 사용자에게 유효한 UI를 보여주지 않는 경우를 방지하는 것**이다.

- state 변수가 역설을 일으키지는 않는지?
- 다른 state 변수에 이미 가은 정보가 담겨있지는 않는지?
- 다른 state 변수를 뒤집었을 때 같은 정보를 얻을 수 있지는 않은지?
  > [Difference between useState and useReducer - GeeksforGeeks](https://www.geeksforgeeks.org/difference-between-usestate-and-usereducer/)

### 5: state 설정을 위해 이벤트 핸들러를 연결하기

모든 상호작용을 state로 표현하게 되면 이후에 새로운 시각적 state가 추가되더라도 기존의 로직이 손상되는 것을 막을 수 있다.

## State 구조 선택하기

### State 구조화 원칙

state의 올바른 구조를 선택하는 데에는 몇 가지 원칙이 있다. 이 원칙은 모두 **오류 없이 상태를 업데이트하는 것을 목표로** 한다.

- 연관된 state 그룹화하기: 둘 이상의 state를 항상 동시에 업데이트 한다면 단일 객체로 병합하는 것을 고려
- State의 모순 피하기
- 불필요한 state 피하기
- State의 중복 피하기: 여러 state 간 동일한 데이터가 중복될 경우 동기화를 유지 하기 어려우므로 중복을 피하라.
- 깊게 중첩된 state 피하기: 깊게 계층화된 state는 업데이트하기 쉽지 않으므로 가능한 평탄하게 구성하라.

### State의 모순 피하기

```javascript
// ❌ 두 state가 서로 모순됨
const [isSending, setIsSending] = useState(false);
const [isSent, setIsSent] = useState(false);

// 🟢 두 state를 status 하나로 대체
const [status, setStatus] = useState("typing");
const isSending = status === "sending";
const isSent = status === "sent";
```

위 두 state는 동시에 `true`일 수 없기 때문에 `setIsSending`과 `setIsSent`를 동시에 호출하는 것을 잊어버리면 버그를 유발한다. 따라서 두 state를 `status`로 대체하는 것이 좋다.

- `isSending`과 같이 가독성을 위해 몇 가지 상수를 선언할 수도 있다.
- 이들은 state 변수가 아니기 때문에 서로 동기화되지 않을 우려는 없다.

### 불필요한 state 피하기

props나 기존 state로 계산할 수 있는 값이라면 상수로 관리하자. 특히 아래의 경우와 같이 **props를 state에 미러링하는 걸 피해야** 한다.

```javascript
function Message({ messageColor }) {
  const [color, setColor] = useState(messageColor);
```

- `color`는 `messageColor`에 의해 `Message` 컴포넌트가 렌더링 될 경우 초기화 된다.
- 문제는, 부모 컴포넌트가 나중에 다른 `messageColor` 값을 전달해줘도 `color`는 업데이트 되지 않는다.

## 컴포넌트 간 State 공유하기

### State 끌어올리기 예제

두 컴포넌트의 state가 항상 함께 변경되기를 원한다면 state를 제거하고 공통의 부모 컴포넌트로 옮긴 후 props로 전달해야 한다. 이 방법을 **State 끌어올리기**라고 한다.

> [What are Controlled and Uncontrolled Components in React.js?](https://www.freecodecamp.org/news/what-are-controlled-and-uncontrolled-components-in-react/)

### 각 state의 단일 진리의 원천

많은 컴포넌트는 자체 state를 갖는다. 일부 state는 입력처럼 리프에 있으며, 일부는 상단에 있다. 각각의 고유한 state에 대해 어떤 컴포넌트가 소유할지 고를 수 있다. 이 원칙은 또한 [단일 진리의 원칙](https://en.wikipedia.org/wiki/Single_source_of_truth)을 갖는다.

- 이는 모든 state가 한 곳에 존재한다는 의미가 아니라 그 정보를 가지고 있는 특정 컴포넌트가 있다는 것
- 컴포넌트 간의 공유된 state를 중복하는 대신 그들의 공통 부모로 끌어올리고 필요한 자식에게 전달해야 됨

## State를 보존하고 초기화하기

각 컴포넌트는 독립된 state를 가지며 React는 UI 트리에서의 위치로 각 state가 어떤 컴포넌트에 속하는지 추적한다. 리렌더링마다 언제 state를 보존하고 또 state를 초기화할지 컨트롤할 수 있다.

### State는 렌더트리의 위치에 연결됩

React는 UI 안에 있는 컴포넌트 구조로 렌더 트리를 만든다. **state는 컴포넌트 안에 있는 것이 아닌 React 안에** 있다.

- React는 트리의 동일 컴포넌트를 동일 위치에 렌더링하는 동안 state를 유지. 언마운트 되었다가 다시 마운트 되면 state는 초기화.

### 같은 자리의 같은 컴포넌트는 state를 보존

이 [예시](https://codesandbox.io/s/6f2zp8?file=/src/App.js&utm_medium=sandpack)에서 `isFancy`가 `true`이든 `false`이든 `<Counter />`는 같은 자리이기 때문에 체크 박스를 선택하거나 선택 해제할 때 카운터 state는 초기화되지 않는다.

<img width="611" alt="스크린샷 2024-07-25 오후 9 44 16" src="https://github.com/user-attachments/assets/028eff4e-fec5-447b-8c8b-764f4ac8c87d">

### 같은 위치의 다른 컴포넌트는 state를 초기화

이 [예시](https://codesandbox.io/s/sxz2lq?file=%2Fsrc%2FApp.js&utm_medium=sandpack)에서 같은 자리의 다른 컴포넌트로 변경된다. 처음에는 `<div>`가 `Counter`를 갖지만 `p`로 바꾸면 React는 UI 트리에서 `Counter`와 그 state를 제거한다.

- 또한 **같은 위치에 다른 컴포넌트를 렌더링할 때 컴포넌트는 그의 전체 서브 트리의 state를 초기화**한다.
- 따라서 리렌더링할 때 state를 유지하고 싶다면 트리 구조가 같아야 한ㄴ다. 구조가 다르면 state 유지가 안된다.

### 같은 위치에서 state를 초기화하기

경우에 따라 같은 위치에서 state를 초기화하고 싶다면, 다른 위치에서 컴포넌트를 렌더링 하거나, 각 컴포넌트에 `key`로 명시적인 식별자를 제공한다.

> [React Re-rendering: Exploring What, Why, and How](https://medium.com/simform-engineering/react-re-rendering-exploring-what-why-and-how-d180d5305892)

```javascript
// 1️⃣은 score가 유지됨
{
  isPlayerA ? <Counter person="Taylor" /> : <Counter person="Sarah" />;
}

//2️⃣은 isPlayerA 변경 마다 score 초기화
{
  isPlayerA && <Counter person="Taylor" />;
}
{
  !isPlayerA && <Counter person="Sarah" />;
}

//3️⃣은 isPlayerA 변경 마다 score 초기화
{
  isPlayerA ? (
    <Counter key="Taylor" person="Taylor" />
  ) : (
    <Counter key="Sarah" person="Sarah" />
  );
}
```

- 1️⃣의 경우, `Counter`가 같은 위치에 나타나기 때문에 React는 `person` props가 변경된 같은 `Counter`로 본다.
- 2️⃣의 경우, `isPlayerA`가 바뀔 때마다 첫번째, 두번째 Counter가 없어졌다 생겼다를 반복하므로 state가 초기화된다.
- 3️⃣의 경우, React에게 `key`라는 명시적인 식별자를 다르게 제공하여 같은 위치임에도 state를 공유하지 않도록 한다.

## state 로직을 reducer로 작성하기

### reducer를 사용하여 state 로직 통합하기

state 업데이트가 여러 이벤트 핸들러로 분산되는 경우 state 업데이트 하는 로직은 reducer를 사용해 컴포넌트 외부로 단일 함수로 통합해 관리할 수 있다.

- state를 설정하는 것에서 `action`을 `dispatch` 함수로 전달하는 것으로 바꾸기
- reducer 함수 작성하기
- 컴포넌트에서 reducer 사용하기
  > [When to use useReducer and when not to use it in React](https://medium.com/@queenskisivuli/when-to-use-usereducer-and-when-not-to-use-it-in-react-f8cd5208aee8)

### reducer 잘 작성하기

- Reducers는 반드시 순수해야 한다. reducer는 렌더링 중에 실행되기 때문이다. (action은 다음 렌더링까지 대기)
  - 입력 값이 같다면 결과 값도 항상 같아야 한다. 사이드 이펙트를 수행하서는 안된다.
- 각 `action`은 데이터 안에서 여러 변경들이 있더라도 하나의 사용자 상호작용을 설명해야 한다.

## Context를 사용해 데이터를 깊게 전달하기

### Context: Props 전달하기의 대안

부모에서 자식 컴포넌트로 props를 전달하는 것은 어떤 경우 Prop drilling 상황을 초래할 수도 있다. 따라서 context를 사용하며 아래 세 단계로 진행된다.

> [Prop Drilling in React Explained with Examples](https://www.freecodecamp.org/news/prop-drilling-in-react-explained-with-examples/)

- Context를 생성
- 데이터가 필요한 컴포넌트에서 context를 사용 (`useContext`)
- 데이터를 지정하는 컴포넌트에서 context를 제공

### 3단계: Context 제공하기

```javascript
export default function Section({ level, children }) {
  return (
    <section className="section">
      <LevelContext.Provider value={level}>{children}</LevelContext.Provider>
    </section>
  );
}
```

- `Provider`는 React에게 `Section` 내의 어떤 컴포넌트가 `LevelContext`를 요구하면 `level`을 주라고 알려준다.
- 컴포넌트는 그 **위에 있는 UI 트리에서 가장 가까운 `<LevelContext.Provider>`의 값을 사용**한다.

### 같은 컴포넌트에서 context를 사용하며 제공하기

Context를 통해 위의 컴포넌트에서 정보를 읽을 수 있으므로 각 `Section`은 위의 `Section`에서 `level`을 읽고 자동으로 `level + 1`을 아래로 전달할 수도 있다.

```javascript
export default function Section({ children }) {
  const level = useContext(LevelContext);
  return (
    <section className="section">
      <LevelContext.Provider value={level + 1}>
        {children}
      </LevelContext.Provider>
    </section>
  );
}
```

## Reducer와 Context로 앱 확장하기

### Reducer와 context를 결합하기

Reducer 상태를 props를 통한 전달 대신 `tasks` state와 `dispatch` 함수를 모두 context에 넣는다면 트리에서 TaskApp **아래에 있는 모든 컴포넌트가 prop drilling 없이 tasks와 dispatch actions를 읽을 수 있다.**

- Context를 생성한다.
- State과 dispatch 함수를 context에 넣는다.
- 트리 안에서 context를 사용한다.
  > [How to combine useContext with useReducer?](https://www.geeksforgeeks.org/how-to-combine-usecontext-with-usereducer/)

이 [예시](https://codesandbox.io/s/jjwhwh?file=/src/TaskList.js&utm_medium=sandpack)와 같이 리팩토링하면 **컴포넌트들이 데이터를 어디서 가져오는지가 아닌 무엇을 보여줄 것인지에 집중**할 수 있도록 깔끔한 정리가 가능해진다.
