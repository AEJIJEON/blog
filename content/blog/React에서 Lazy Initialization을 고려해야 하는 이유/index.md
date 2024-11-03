---
title: React에서 Lazy Initialization을 고려해야 하는 이유
date: "2024-11-02T15:42:42.746Z"
description: "lazy initialization이 무엇인지, React에서 어떻게 활용할 수 있는지, 그리고 이를 적용하여 문제를 해결하는 과정을 살펴보겠습니다."
---
다음은 `Promise` 객체와 해당 객체의 `resolve` 함수를 `ref`로 관리하려는 코드입니다.

```jsx
function Component(){
  const saveResolve = useRef<(data: Data) => void>();
  const savePromise = useRef<Promise<Data>>(
    new Promise((resolve) => {
      saveResolve.current = resolve;
    }),
  );
  // ...
}
```

이 코드에서 어떤 문제가 발생할 수 있을까요?

Promise가 `resolve`된 후에도 `savePromise`에 접근하면 계속 `pending` 상태가 유지되는 이슈가 생깁니다. 원인은 **모든 렌더링마다** `new Promise(...)`가 호출되어 `saveResolve`가 매번 새로운 `resolve` 함수를 할당받기 때문입니다. 이 문제를 해결하기 위해 `lazy initialization`을 적용했습니다. 이번 글에서는 lazy initialization이 무엇인지, React에서 어떻게 활용할 수 있는지, 그리고 이를 적용하여 문제를 해결하는 과정을 살펴보겠습니다.

## Lazy Initialization이란?

[lazy initialization](https://ko.wikipedia.org/wiki/지연된_초기화)은 초기화가 무거운 작업이거나 메모리를 많이 차지하는 경우, **값의 초기화를 필요한 시점까지 지연**시키는 기법입니다. 이를 통해 프로그램의 시작 속도를 개선하거나 메모리 사용을 최적화할 수 있습니다.

**예시 코드**:

```jsx
class LazyLoader {
  constructor() {
    this._heavyResource = null;  // 초기에는 null로 설정 (지연 초기화)
  }

  get heavyResource() {
    // heavyResource가 아직 초기화되지 않은 경우에만 초기화
    if (!this._heavyResource) {
      console.log("Heavy resource initializing...");
      this._heavyResource = new HeavyResource(); // 필요한 시점에 생성
    }
    return this._heavyResource;
  }
}

class HeavyResource {
  constructor() {
    console.log("HeavyResource created!");
    // 실제로 무거운 초기화 작업이 여기에 들어간다고 가정
  }

  doSomething() {
    console.log("Doing something with HeavyResource.");
  }
}

// 사용 예시
const loader = new LazyLoader();

console.log("Before accessing heavyResource...");
loader.heavyResource.doSomething();  // 이때 heavyResource가 처음 생성됨
console.log("After accessing heavyResource...");
```

## React에서의 Lazy Initialization

React에서도 여러 상황에서 lazy initialization을 활용할 수 있습니다.

1. **상태 초기화 시**
    
    상태 초기화는 기본적으로 초기 렌더링 시점에 이뤄지지만, [함수형 초기화](https://ko.react.dev/reference/react/useState#avoiding-recreating-the-initial-state)를 통해 **최초 렌더링 시점에만 함수가 실행되도록** 보장할 수 있습니다. 이는 초기화 과정에서 비용이 큰 작업일 때 특히 유용합니다.
    
    ```jsx
    const [state, setState] = useState(createInitialValue);
    ```
    
2. `useRef`를 이용한 값 초기화
    
    useRef에서도 함수형 초기화를 통해 lazy initialization을 수행할 수 있을까요? 불가능합니다.
    useRef 파라미터에 함수를 넘기면 단순히 해당 함수를 참조하는 object가 생성됩니다.
    
    ```jsx
    function Component(){
    	const ref = useRef(() => null);
    	console.log(ref) // {current: ƒ}, current : () => null
    // ...
    }
    ```
    
    하지만, 다음과 같은 방식으로 lazy initialization을 수행할 수 있습니다.
    
    ```jsx
    function Component() {
      const ref = useRef(null);
      if (ref.current === null) {
        ref.current = createInitialValue();
      }
      // ...
    }
    ```
    
    [react에서도 이 방법을 제안하고 있습니다.](https://ko.react.dev/reference/react/useRef#avoiding-recreating-the-ref-contents)
    
    또는 필요한 시점에만 초기화되도록 다음과 같은 방식을 사용할 수도 있습니다.
    
    ```jsx
    function Component() {
      const ref = useRef(null);
    
      function getInstance() {
        let instance = ref.current;
        if (instance !== null) return instance;
        ref.current = createInitialValue();
        return ref.current;
      }
    
      const instance = getInstance();
      // ...
    }
    ```
    
    이처럼 `ref.current`에 직접 접근하지 않고, **접근 메서드**를 만들어 lazy initialization을 구현할 수 있습니다.
    
3. `useMemo`로 lazy initialization 구현
    
    `useMemo`를 사용해  lazy initialization을 구현할 수도 있습니다. 
    

    ```jsx
    const initialValue = useMemo(() => createInitialValue(), []);
    const [state, setState] = useState(initialValue);
    ```

    다만, 추가 메모리 할당이 필요하며 React에서 제안하는 lazy initialization 방식들이 더 간단하고 직관적입니다.

## 문제 해결하기

앞서 언급한 `promise/resolve` 문제는 모든 렌더링에서 호출되는 `promise` 생성으로 인해 발생했습니다. 다음과 같이 lazy initialization을 적용하여 문제를 해결했습니다.

```jsx

function Component() {
  const saveResolve = useRef<(data: Data) => void>();
  const savePromise = useRef<Promise<Data> | null>(null);

  if (savePromise.current === null) {
    savePromise.current = new Promise((resolve) => {
      saveResolve.current = resolve;
    });
  }
  // ...
}

```

이제 `savePromise`는 초기 렌더링 시에만 생성되며, 그 이후에는 생성된 `resolve` 함수가 유지됩니다. lazy initialization을 적용함으로써 불필요한 `promise` 생성을 방지하고, 필요한 시점에서만 초기화되는 구조를 갖출 수 있었습니다.

정리하면, Lazy initialization을 적절히 사용하면 성능을 최적화할 수 있을 뿐만 아니라 불필요한 반복 작업에서 발생할 수 있는 문제를 사전에 방지할 수 있기 때문에 코드를 개선할 때 고려해보면 좋을 것 같습니다.

## 참고 자료
- [https://ko.wikipedia.org/wiki/지연된_초기화](https://ko.wikipedia.org/wiki/%EC%A7%80%EC%97%B0%EB%90%9C_%EC%B4%88%EA%B8%B0%ED%99%94)
- [https://ko.react.dev/reference/react/useState#avoiding-recreating-the-initial-state](https://ko.react.dev/reference/react/useState#avoiding-recreating-the-initial-state)
- [https://ko.react.dev/reference/react/useRef#avoiding-recreating-the-ref-contents](https://ko.react.dev/reference/react/useRef#avoiding-recreating-the-ref-contents)
- [https://github.com/facebook/react/issues/14490#issuecomment-454973512](https://github.com/facebook/react/issues/14490#issuecomment-454973512)