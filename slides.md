---
title: Slides Template
separator: <!--s-->
verticalSeparator: <!--v-->
theme: black
revealOptions:
    transition: 'fade'
---

## Asynchronous React

Kartseva Anastasia

Aston Lab

<!--s-->
#### План доклада

1. Очень кратко об асинхронности в JS

2. Асинхронность в реализации React Hooks

  *  Хук useState и асинхронное обновление состояния
  * Асинхронная функция внутри хука useEffect

3. Обработка асинхронных запросов в React

4. Конкуррентный режим и React.Suspense
<!--s-->

### Event Loop

<img src="images/eventloop.png" alt='Event loop image'/>

<!--s-->

### useState

```
const MyComponent = () => {
  const [state, setState] = useState('prev');
  const handleClick = () => {
    setState('next');
    console.log(state); // 'prev'
  };

  // setState is async!

  useEffect(() => {
    console.log(state); // 'next'
  }, [state]);
};

```

<!--s-->
### Batch of state updates

```
export default function App() {
  const [counter1, setCounter1] = useState(0); // 1
  const [counter2, setCounter2] = useState(0); // 1
  const [counter3, setCounter3] = useState(0); // 1
  const [renderCount, setRenderCount] = useState(0); // 1

  useEffect(() => {
    setRenderCount(renderCount + 1);
  }, [counter1, counter2, counter3]);

  const handleClick = () => {
    setCounter1(counter1 + 1);
    setCounter2(counter2 + 1);
    setCounter3(counter3 + 1);
  }

  return (
    <div className='App'>
      <h1>Function Component</h1>
      <div>
        Counter1: {counter1}
      </div>
      <div>
        Counter2: {counter2}
      </div>
      <div>
        Counter3: {counter3}
      </div>
      <br/>
      <div>Component was rendered {renderCount} times</div>
      <button onClick={handleClick}>Click me</button>
    </div>
  );
}

```

<!--s-->
### React 18: Automatic Batching

```
  async function handleChange(evt) {
    const name = evt.target.value;
    const response = await fetch('//some.url', JSON.stringify({ value })) // Api call that calculates the length and returns the value
    setName(v => name) // Does not re-render
    setLength(c => response.value) // Does not re-render
  }
```

<!--s-->
### useEffect callback: incorrect

```
  // Warning: Effect callbacks are synchronous to prevent race conditions

  useEffect(async () => {
    const products = await fetch(`${API_URL}/products.json`);
    setProducts(products);
  }, []);
```
<!--s-->

### useEffect callback: correct

```
  useEffect(() => {
    const fetchData = async () => {
      const products = await fetch(`${API_URL}/products.json`);
      setProducts(products);
    });

    fetchData();
  }, []);

  OR
  // IIFE
  useEffect(() => (async () => {
    const products = await fetch(`${API_URL}/products.json`);
      setProducts(products);
    })();
  }, []);

```

<!--s-->
#### Handling async requests in React

```
  const [items, setItems] = useState([]); // empty array
  useEffect(
    // ... async request and state update
  )

  return (
    <div>
      {items.map(...)} // renders nothing
    </div>
  )
```

<!--s-->

#### Handling async requests in React
###### “fetch-on-render”
```
  const [items, setItems] = useState([]);
  const [isLoading, setIsLoading] = useState(true);
  useEffect(() => {
    // ... async request and state update
    setIsLoading(false);
  }, []);

  return (
    <div>
      {isLoading ? <p>Загрузка...</p> : items.map(...) }
    </div>
  )
```

<!--s-->
#### Handling async requests in React
##### “fetch-on-render”
```
  const [loadState, setLoadState] = useState('idle');

  useEffect(() => {
    setLoadState('loading');
    const fetchData = async () => {
      try {
        const data = await fetch(...);
        setLoadState('success');
      } catch (err) {
        setLoadState('fail');
      }
    };
    fetchData();
  }, []);

```
<!--s-->
```
  ...
  const renderContent = () => {
    switch(loadState) {
      case 'loading':
        return <p>Загрузка...</p>;
      case 'fail':
        return <p>Что-то пошло не так!</p>;
      case 'success':
        return items.map(...);
      case 'idle':
      default:
        return null;
    }
  }

  return (
    <div>{ renderContent() }</div>
  );
```
<!--s-->
### React.Suspense
### waiting for promise to resolve

```
  function handleClick() {
    setTab('comments'); // setter is async
  }

  <Suspense fallback={<Spinner />}>
    {tab === 'photos' ? <Photos /> : <Comments />}
  </Suspense>

```
<!--s-->

### React.Suspense
### Concurrent mode 

  ```
  const [isPending, startTransition] = useTransition();

  function handleClick() {
    startTransition(() => {
      setTab('comments');
    });
  }

  <Suspense fallback={<Spinner />}>
    <div style={{ opacity: isPending ? 0.8 : 1 }}>
      {tab === 'photos' ? <Photos /> : <Comments />}
    </div>
  </Suspense>

  ```
<!--s-->
### React.Suspense for data fetching
#### Render-as-you-fetch

future React updates
```
  const todos = fetchData('/todos')

  return (
    <Suspense fallback={<Spinner />}>
      <Todos data={todos} />
    </Suspense>
  ) // tracking async request

```
<!--v-->

### Links
* [Async useState](https://dev.to/fidalmathew/usestate-is-asynchronous-learn-how-to-use-usestate-and-useeffect-properly-1m1m)
* [Automnatic Batching](https://blog.devgenius.io/react-18-automatic-batching-2f5d691b4f19)
* [useEffect callback](https://www.designcise.com/web/tutorial/why-cant-react-useeffect-callback-be-async)
* [useEffect callback](https://www.designcise.com/web/tutorial/does-javascript-async-function-implicitly-return-a-promise)
* [Async requests handling](https://www.clearpeople.com/blog/how-to-handle-asynchronous-requests-gracefully-in-react)
* [Concurrent Mode in React](https://www.telerik.com/blogs/concurrent-mode-react)
* [React.Suspense new features](https://blog.logrocket.com/react-suspense-data-fetching/)
