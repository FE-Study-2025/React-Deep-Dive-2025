
# 05장 리액트와 상태 관리 라이브러리

## 상태 관리는 왜 필요한가?
<b>상태</b>: 어떤 의미를 지닌 값. 애플리케이션 내부 시나리오에 따라 지속적으로 변경될 수 있는 값.

예시: 
1. 화면상의 상호 작용이 가능한 모든 요소의 현재 값. (다크/라이트 모드, input, 모달 노출 여부)
2. url queryString, params, hash
3. form 상태 (Loading, submit, disabled, validation)
4. api 요청을 통해 서버에서 가져온 값

그러나 애플리케이션 전체적으로 관리해야 할 상태가 있어야 하고, 다양한 요소들이 각 상태에 맞는 UI를 보여줘야 한다고 가정했을 때,이러한 상태를 효율적으로 관리하고, 상태가 필요한 쪽에서는 빠르게 반응할 수 있는 모델에 대한 고민이 필요하다.

### Flux 패턴

Flux 패턴은 상태 관리 라이브러리의 기본 아키텍처이다.

단방향으로 데이터 흐름을 가지는 Flux 패턴은 다음과 같은 요소로 이루어져 있다.
1. Action: 어떠한 작업을 처리할 액션과 그 액션 발생 시 함께 포함시킬 데이터를 의미. 액션 타입과 데이터를 각각 정의해 이를 디스패처로 보낸다.
2. Dispatcher: 액션을 스토어에 보내는 역할.
3. Store: 상태를 관리하는 스토어. 액션의 타입에 따라 어떻게 상태를 변경할지 정의.
4. View: 스토어에서 만들어진 데이터를 가져와 화면을 렌더링하는 역할을 한다. 또한 사용자의 인터렉션에 따라 뷰에서 액션을 발생시킨다.

이러한 Flux 패턴의 요소들을 직접 작성해야 하므로 코드의 양이 많아지지만, 데이터 흐름이 단방향이기 때문에 데이터 흐름을 쉽게 파악할 수 있다.

### 리덕스의 등장
- 리덕스는 하나의 상태 객체를 스토어에 저장해 두고, 이 객체를 업데이트하는 action을 정의한다.
- 이러한 action을 디스패처에 보내면, 디스패처는 이 action을 스토어에 전달한다.
- 스토어는 이 action을 받아 상태 객체를 업데이트한다.
- 이렇게 업데이트된 상태 객체는 뷰에 전달되어 화면을 렌더링한다.

이러한 리덕스의 특징은 다음과 같다.
- 하나의 글로벌 상태 객체를 통해 하위 컴포넌트에 전파할 수 있어 props drilling 문제를 해결할 수 있다.
- 스토어가 필요한 컴포넌트라면 단지 connect만 쓰면 스토어에 접근할 수 있다.
- 그러나 액션 타입을 정의하고, 액션을 수행하는 creator 함수, dispatcher, selector 등 많은 보일러플레이트 코드를 작성해야 한다.

### Context API와 useContext 
- 리덕스의 무거운 구조 대신, 전역 상태를 하위 컴포넌트에 주입할 수 있는 Context API가 등장했다.

### 훅의 탄생, 그리고 React Query와 SWR, Recoil, Zustand, Jotai, Valtio에 이르기까지
- 리액트 16.8 버전에서 함수형 컴포넌트에 사용할 수 있는 다양한 훅 API가 추가되었다.
- 이러한 훅을 통해 API 호출에 대한 상태를 관리하는 HTTP 요청에 특화된 상태 관리 라이브러리 (React Query와 SWR),좀 더 범용적으로 쓸 수 있는 상태 관리 라이브러리 (Recoil, Zustand, Jotai, Valtio)가 등장했다.


## 리액트 훅으로 시작하는 상태 관리
### useState, useReducer
- useState, useReducer 모두 지역 상태 관리를 위해 만들어졌다. 이 지역 상태는 해당 컴포넌트 내에서만 유효하다.
- 이 두 훅을 사용할 때마다 컴포넌트별로 초기화되므로 컴포넌트에 따라 서로 다른 상태를 가질 수밖에 없다.

### 지역 상태를 벗어나는 전역 상태 관리 store
전역 상태를 참조하고 이를 통해 렌더링까지 자연스럽게 이루어지는 store의 조건은 다음과 같다.
1. 컴포넌트 외부 어딘가에 상태를 두고 여러 컴포넌트가 이 상태를 참조할 수 있어야 한다.
2. 이 외부에 있는 상태를 사용하는 컴포넌트는 상태의 변화를 알아챌 수 있어야 하고, 상태가 변했을 때 리렌더링이 일어나서 컴포넌트를 최신 상태값 기준으로 렌더링할 수 있어야 한다.
3. 상태가 원시값이 아닌 객체인 경우에 그 객체에 내가 감지하지 않는 값이 변한다 하더라도 리렌더링이 발생해서는 안 된다.

```tsx
import React, { useState, useEffect, useCallback, useMemo, useSubscription } from 'react';

type Initializer<T> = T extends any ? T | ((prev: T) => T) : never;

type State<T> = T extends any ? T : never;

type Action<T> = T extends any ? ((prev: State<T>) => State<T>) : never;

type Store<State> = {
    get: () => State;
    set: (action: Initializer<State>) => State;
    // store의 값이 변경될 때마다 변경되었을을 알리는 callback 함수를 실행해야 하고, 이 callback을 등록할 수 있는 subscribe 함수가 필요하다.
    subscribe: (callback: () => void) => () => void;
};

export const createStore = <State extends unknown>(initialState: Initializer<State>): Store<State> => { 
    let state = typeof initialState === 'function' ? initialState() : initialState;
    
    // createStore는 자신이 관리해야 하는 상태를 내부 변수로 가진 다음, 
    const callbacks = new Set<() => void>();

    // get 함수로 해당 변수의 최신값을 제공하며, 
    const get = () => state;
    // set 함수로 내부 변수를 최신화하며, 이 과정에서 등록된 콜백을 모조리 실행한다.
    const set = (nextState: State | ((prev: State) => State)) => {
        state = typeof nextState === 'function' ? nextState(state) : nextState;

        // 값의 설정이 발생하면 콜백 목록을 순회하면서 모든 콜백을 실행한다.
        callbacks.forEach((callback) => callback());

        return state;
    };

    // subscribe 함수로 콜백을 등록할 수 있고, 이 함수는 콜백을 등록하고 해당 콜백을 삭제 할 수 있는 함수를 반환한다.
    const subscribe = (callback: () => void) => {
        callbacks.add(callback);
        return () => callbacks.delete(callback);
    };

    return {
        get,
        set,
        subscribe,
    };
};

// createStore로 만든 store의 변화를 감지하는 사용자 정의 훅. store 값의 변화에 따라 컴포넌트 렌더링을 유도한다.

// 1. 먼저 훅의 인수로 사용할 store를 받는다.
export const useStore = <State extends unknown>(store: Store<State>) => {
    // 2. store의 값으로 초기화를 한다. 이제 이 useState가 컴포넌트의 렌더링을 유도한다.
    const [state, setState] = useState<State>(() => store.get());

    // 3. store의 현재 값을 가져와 setState를 수행하는 함수를 store.subscribe에 등록한다. 
    // createStore 내부에서 값이 변경될 때마다 subscribe에 등록된 함수를 실행하므로 useStore 내부에서는 store의 값이 변경될 때마다 state값이 변경되는 것을 보장받을 수 있다.
    useEffect(() => {
        const unsubscribe = store.subscribe(() => setState(store.get()));
        return unsubscribe;
    }, [store]);

    return [state, store.set] as const;
};

const store = createStore({count: 0});

function Counter() {
    const [state, setCount] = useStore(store);

    const handleClick = () => {
        setCount((prev) => ({count: prev.count + 1}));
    }

    return (
        <>
            <p>{state.count}</p>
            <button onClick={handleClick}>+</button>
        </>
    )
}
```

위 코드에서 원하는 값이 변했을 때만 리렌더링되도록 훅을 다시 구성해보자.
```tsx
// 리액트 외부에서 관리되는 값에 대한 변경을 추적하고, 이를 리렌더링까지 할 수 있는 useStoreSelector 훅을 만들어보자.
export const useStoreSelector = <State extends unknown, Value extends unknown>(
    store: Store<State>,
    // store의 상태에서 어떤 값을 가져올지 정의하는 selector 함수
    selector: (state: State) => Value,
) => {
    const [state, setState] = useStore(() => selector(store.get()));

    useEffect(() => {
        const unsubscribe = store.subscribe(() => setState(selector(store.get())));
        return unsubscribe;
    }, [store, selector]);
    
    return state;
};

const store2 = createStore({count: 0, text: 'Hello'});

function Counter2() {
    // useStoreSelector에 제공하는 2번째 인수 selector함수는 useCallback을 사용해 참조 값이 변하지 않도록 하거나,
    const count = useStoreSelector(store2, useCallback((state) => state.count, []));

    function handleClick() {    
        store2.set((prev) => ({  ...prev, count: prev.count + 1}));
    }

    useEffect(() => {
        console.log('Counter2 rendered');
    });

    return (
        <>
            <p>{count}</p>
            <button onClick={handleClick}>+</button>
        </>
    )
}
// 컴포넌트 밖에 선언하여 컴포넌트가 리렌더링될 때마다 함수가 계속 재생성되어 store의 subscribe 를 반복적으로 수행하는 것을 막아야 한다.
const textSelector = (state: ReturnType<typeof store2.get>) => state.text;

function TextEditor() {
    const text = useStoreSelector(store2, textSelector);

    const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
        store2.set((prev) => ({...prev, text: e.target.value}));
    }

    useEffect(() => {
        console.log('TextEditor rendered');
    });

    return (
        <>
            <p>{text}</p>
            <input type="text" value={text} onChange={handleChange} />
        </>
    )
}

function NewCounter() {
    const subscription = useMemo(() => ({
        getCurrentValue: () => store.get(),
        subscribe: (callback: () => void) => {
            const unsubscribe = store.subscribe(callback);
            return unsubscribe;
        },
    }), []);

    // useStoreSelector나 useStore를 사용했던 것과 마찬가지로 리액트에서 제공하는 useSubscription 훅은 리액트 외부에서 관리되는 값에 대한 변경을 추적하고, 이를 리렌더링까지 할 수 있는 훅이다. 
    const value = useSubscription(subscription);

    return <>{JSON.stringify(value)}</>
}
```


#### useSubscription을 타입스크립트로 구현한 코드다.
```tsx
import React, { useState, useEffect, useDebugValue } from 'react';

export function useSubscription<Value>({
    getCurrentValue,
    subscribe,
}: {
    getCurrentValue: () => Value;
    subscribe: (callback: () => void) => () => void;
}): Value {
    const [state, setState] = useState<Value>({
        getCurrentValue,
        subscribe,
        value: getCurrentValue(),
    });

    // 현재 가져올 값을 따로 변수에 저장해 둔다.
    let valueToReturn = state.value;

    if(state.getCurrentValue !== getCurrentValue || state.subscribe !== subscribe) {
        valueToReturn = getCurrentValue();
        setState({
            getCurrentValue,
            subscribe,
            value: valueToReturn,
        });
    }

    useDebugValue(valueToReturn);

    useEffect(() => {
        let didUnsubscribe = false;

        const checkForUpdates = () => {
            if(didUnsubscribe) return;

            const value = getCurrentValue();
            
            setState((prev) => {
                if(prev.getCurrentValue !== getCurrentValue || prev.subscribe !== subscribe) {
                    return prev;
                }

                if(prev.value === value) {
                    return prev;
                }

                return {
                    ...prev,
                    value,
                };
            });
        };

        const unsubscribe = subscribe(checkForUpdates);

        checkForUpdates();

        return () => {
            didUnsubscribe = true;
            unsubscribe();
        };
    }, [subscribe, getCurrentValue]);

    return valueToReturn;
}
```



### useState와 Context를 동시에 사용해 보기
만약 useStore 내지는 useStoreSelector 훅을 사용하는 서로 다른 스코프에서 스토어의 구조는 동일하되, 여러 개의 서로 다른 데이터를 공유해 사용하고 싶다면 Context를 활용해 해당 스토어를 하위 컴포넌트에 주입하여 사용할 수 있다. 

각각의 컴포넌트에서는 자신이 주입된 스토어에 대해서만 접근할 수 있게 된다.
```tsx
interface CounterStore {
    count: number;
    text: string;
}
// Context를 생성하면 자동으로 스토어도 함께 생성한다.
export const CounterStoreContext = createContext<CounterStore>({count: 0, text: 'hello'});

export const CounterStoreProvider = ({initialState, children}: PropsWithChildren<{initialState: CounterStore}>) => {
    const storeRef = useRef<Store<CounterStore>>();

    if(!storeRef.current) {
        storeRef.current = createStore(initialState);
    }

    return (
        <CounterStoreContext.Provider value={storeRef.current}>
            {children}
        </CounterStoreContext.Provider>
    );
}

export const useCounterContextSelector = <State extends unknown>(selector: (state: CounterStore) => State) => {
    const store = useContext(CounterStoreContext);

    const subscription = useSubscription(useMemo(() => ({
        getCurrentValue: () => selector(store.get()),
        subscribe: store.subscribe,
    }), [store, selector]));

    return [subscription, store.set] as const;
}

const ContextCounter = () => {
    const id = useId();
    const [counter, setCounter] = useCounterContextSelector(useCallback((state) => state.count, []));

    const handleClick = () => {
        setCounter((prev) => ({...prev, count: prev.count + 1}));
    }

    useEffect(() => {
        console.log(`${id} rendered`);
    });

    return (
        <div>
            <p>{counter}</p>
            <button onClick={handleClick}>+</button>
        </div>
    );
}

const ContextInput = () => {
    const id = useId();
    const [text, setStore] = useCounterContextSelector(useCallback((state) => state.text, []));

    const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
        setStore((prev) => ({...prev, text: e.target.value}));
    }

    useEffect(() => {
        console.log(`${id} rendered`);
    });

    return (
        <div>
            <input type="text" value={text} onChange={handleChange} />
        </div>
    );
}

export default function App() {
    return (
        <>
            {/* 0 */}
            <ContextCounter />
            {/* hello */}
            <ContextInput />
            <CounterStoreProvider initialState={{count: 10, text: 'hi'}}>
                {/* 10 */}
                <ContextCounter />
                {/* hi */}
                <ContextInput />
                <CounterStoreProvider initialState={{count: 20, text: 'welcome'}}>
                    {/* 20 */}
                    <ContextCounter />
                    {/* welcome */}
                    <ContextInput />
                </CounterStoreProvider>
            </CounterStoreProvider>
        </>
    );
}
```
Context와 Provider를 기반으로 각 store값을 격리해서 작성한 코드의 장점은 다음과 같다.
- 각 컴포넌트가 사용하는 상태가 어느 스토어에서 온 상태인지 신경쓰지 않아도 된다.
- 부모 자식 컴포넌트의 책임과 역할을 이름이 아닌 Context와 Provider로 나눌 수 있어 코드 작성이 한결 용이해 진다.




## 상태관리 라이브러리 살펴보기 - Zustand

- Zustand는 리덕스에 영감을 받아 만들어졌다.
- 하나의 스토어를 중앙 집중형으로 활용해 이 스토어 내부에서 상태를 관리하고 있다.
- zustand의 store는 리액트를 비롯한 그 어떤 프레임워크와는 별개로 완전히 독립적으로 구성되어 있다.
- 따라서 순수 자바스크립트 환경에서도 사용할 수 있다.

```tsx
type TState = ReturnType<typeof createState>;
type Listener = (state: TState, prevState: TState) => void;

const createStoreImpl: CreateStoreImpl = (createState) => {
    let state: TState;
    const listeners: Set<Listener> = new Set();

    const setState: SetStateInternal<TState> = (partial, replace) => {
        // state의 일부만 변경하고 싶을 때 partial을 사용
        const nextState = typeof partial === 'function' ? partial(state) : partial;

        if(nextState !== state) {
            const previousState = state;
            // 완전히 새로운 객체를 할당하고 싶을 때 replace를 사용
            state = replace ?? typeof nextState !== 'object ? nextState : Object.assign({}, state, nextState);
            listeners.forEach((listener) => listener(state, previousState));
        }
    }
    // 클로저의 최신 값을 가져오기 위해 함수로 만들어져 있다.
    const getState: () => TState = () => state;

    const subscribe: (listener: Listener) => () => void = (listener) => {
        listeners.add(listener);
        return () => listeners.delete(listener);
    }

    const destroy: () => void = () => listeners.clear();
    const api = {getState, setState, subscribe, destroy};
    state = (createState as PopArgument<typeof createState>)(setState, getState, api);

    return api as any;
}   
```

```tsx
type CounterStore = {
    count: number;
    increase: (num: number) => void;
}

// set이라는 인수로 상태를 변경하는 함수를 받아온다. createStoreImpl에서 createState를 호출할 때 인수로 전달된다.
const store = createStore<CounterStore>((set) => ({
    count: 0,
    increase: (num: number) => set((state) => ({count: state.count + num})),
}));

store.subscribe((state, prev) => {
    if (state.count !== prev.count) {
        console.log(`count has been changed`, state.count);
    }
});

store.setState((state) => ({ count: state.count + 1 }));

store.getState().increase(10);
```

#### zustand의 useStore 구현
```tsx
export function useStore<TState, StateSlice>(
    api: WithReact<StoreApi<TState>>,
    selector: (state: TState) => StateSlice = api.getState as any,
    equalityFn?: (a: StateSlice, b: StateSlice) => boolean,
){
    // useSyncExternalStoreWithSelector는 useSyncExternalStore( react18 new hook )의 확장 버전으로 원하는 값을 가져올 수 있는 selector와 동등비교를 할 수 있는 equalityFn 함수를 받는다는 차이가 있다.
    const slice = useSyncExternalStoreWithSelector(
        api.subscribe,
        api.getState,
        api.getServerState || api.getState,
        selector,
        equalityFn,
    );

    useDebugValue(slice);
    return slice;
}
```

#### zustand의 리액트에서 사용할 수 있는 스토어를 만들어주는 변수인 create. 
```tsx
const createImpl = <T>(createState: StateCreator<T, [], []>) => {
    const api = typeof createState === 'function' ? createStore(createState) : createState;

    const useBoundStore: any = (selector?: any, equalityFn?: any) => useStore(api, selector, equalityFn);

    // useBoundStore에 api의 메서드를 할당해 즉시 컴포넌트에서 api를 사용할 수 있게 한다.
    Object.assign(useBoundStore, api);

    return useBoundStore;
};

const create = (<T>(createState: StateCreator<T, [], []> | undefined) => createState ? createImpl(createState) : createImpl) as CreateStore;

export default create;
    
```


```tsx
interface Store {
    count: number;
    text: string;
    increase: (num: number) => void;
    setText: (text: string) => void;
}

// createStore는 리액트와 상관없는 바닐라 스토어를 만들 수 있으며, useStore 훅을 통해 접근해 리액트 컴포넌트 내부에서 사용할 수 있게 된다.
const store = createStore<Store>((set) => ({
    count: 0,
    text: '',
    increase: (num) => set((state) => ({count: state.count + num})),
    setText: (text) => set((state) => ({text})),
}));

const counterSelector = ({count, increase}: Store) => ({
    count,
    increase,
});

function Counter() {
    const {count, increase} = useStore(store, counterSelector);

    function handleClick() {
        increase(1);
    }

    return (
        <div>
            <p>{count}</p>
            <button onClick={handleClick}>+</button>
        </div>
    )
}

const inputSelector = ({text, setText}: Store) => ({
    text,
    setText,
});

function Input() {  
    const {text, setText} = useStore(store, inputSelector);

    function handleChange(e: React.ChangeEvent<HTMLInputElement>) {
        setText(e.target.value);
    }

    useEffect(() => {
        console.log('Input rendered');
    });

    return (
        <div>
            <input type="text" value={text} onChange={handleChange} />
        </div>
    )
}
```

#### 간단한 사용법
zustand의 create 함수는 컴포넌트 내부에서 사용할 수 있는 훅을 반환한다.
```tsx
import { create } from 'zustand';

const useCounterStore = create<CounterStore>((set) => ({
    count: 1,
    increase: () => set((state) => ({count: state.count + 1})),
    decrease: () => set((state) => ({count: state.count - 1})),
}));

function Counter() {
    const {count, increase, decrease} = useCounterStore();

    return (
        <div className="counter">
            <p>{count}</p>
            <button onClick={increase}>+</button>
            <button onClick={decrease}>-</button>
        </div>
    );
}

```
또, 리액트 외부에 store를 만드는 것도 가능하다.
createStore는 리액트와 상관없는 바닐라 스토어를 만들 수 있으며, useStore 훅을 통해 접근해 리액트 컴포넌트 내부에서 사용할 수 있게 된다.

```tsx
import { createStore, useStore } from 'zustand';

const store = createStore<CounterStore>((set) => ({
    count: 1,
    increase: () => set((state) => ({count: state.count + 1})),
    decrease: () => set((state) => ({count: state.count - 1})),
}));

function Counter() {
    const {count, increase, decrease} = useStore(store);

    return (
        <div className="counter">
            <p>{count}</p>
            <button onClick={increase}>+</button>
            <button onClick={decrease}>-</button>
        </div>
    );
}

```

### Zustand의 특징
- 특별히 많은 코드를 작성하지 않아도 빠르게 스토어를 만들고 사용할 수 있다는 장점이 있다.
- 고작 2.9kB 크기로, api가 복잡하지 않고 사용이 간단해 쉽게 접근할 수 있다는 장점이 있다.
- 타입스크립트 기반으로 작성되어 있어 타입 안정성을 보장받을 수 있다.
- 리덕스와 마찬가지로 미들웨어를 지원하여 기본적인 작동 외에 추가적인 작업 (예: sessionStorage에 저장, 복잡한 객체관리)을 할 수 있다.

## 상태 관리 라이브러리 선택 시 고려해야 할 사항
- 리액트에서 리렌더링을 일으키기 위한 방식은 제한적이기 때문에 어떠한 방식으로 상태를 관리하든지 간에 리렌더링을 만드는 방법은 거의 동일하다.
- npm에서 제공하는 모든 라이브러리와 마찬가지로 메인테이너가 많고 다운로드가 많고 이슈관리가 잘되고 있는 라이브러리를 선택하는 것이 좋다.
- 향후 리랙트의 방향성에 따라 원활한 대응이 가능해야 애플리케이션의 장기적인 유지보수 및 개선이 용이해 진다.

