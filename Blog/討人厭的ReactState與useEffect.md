# 討人厭的 React State 與 useEffect

useEffect 是許多寫 React 的人必須要學習的 Hook 之一，它也是 React 中非常重要的概念之一。也讓很多人又愛又恨，愛的是它讓我們可以輕鬆的處理非同步的資料，恨的是它讓我們在寫程式時，常常會遇到一些難以理解的問題。

我相信很多人在看官方文件時心裡一定覺得「這是什麼鬼？」，官方文件寫得非常抽象，讓人難以理解。而且文鄒鄒的，讓人看了就頭痛，閱讀起來也不知道在講什麼。我今天針對我寫 React 專案總結的經驗來說明 useEffect 的運作方式，還有各種應用場景你可以怎麼解決。

> 「更新 Callback Function 中的狀態。」

這句話貫穿我所認知的 useEffect。

## useEffect Core Concept

官方大多都是這樣寫，但其實背後要傳達的底層邏輯可能跟你想像的有一點落差。但確實這樣寫是對的，只是你還沒理解到背後的運作原理。大部分你也不會遇到問題。

```js
function App({ children }) {
  const [count, setCount] = useState(0);
  const ref = useRef();

  useEffect(() => {
    const onClick = () => {
      console.log("onClick", count);
    };
    const element = ref.current;
    element.addEventListener("click", onClick);
    return () => {
      element.removeEventListener("click", onClick);
    };
  }, [count]);

  return <a ref={ref}>{children}</a>;
}
```

### 來一個 setTimeout 的例子

```js
function App({ children }) {
  const [count, setCount] = useState(0);
  useEffect(() => {
    const id = setTimeout(() => {
      console.log("Hello World", count);
    }, 1000);
    return () => {
      clearTimeout(id);
    };
  }, [count]);

  const onClick = () => {
    setCount(count + 1);
  };

  return <button onClick={onClick}>{children}</button>;
}
```

確實 `count` 改變時，`useEffect` 會執行，但 `setTimeout` 的 callback 卻不會馬上執行，而是等到 `setTimeout` 的時間到了才會執行。所以在 `setTimeout` 時間到之前，`count` 改變了，callback 執行時，`count` 的值還是舊的。所以你需要把 `count` 的值傳入 `useEffect` 的 dependency array 中，這樣當 `count` 改變時，`useEffect` 會重新執行，`setTimeout` 的 callback 也會被清除，但那個 `1000ms` 卻被重新計算。

### Simple Solution

如果說希望更新 State ，卻不希望重新計算 `setTimeout` 的時間，你可以使用 `useRef` 來解決。

```js
function App({ children }) {
  const [count, setCount] = useState(0);
  const ref = useRef(0);
  useEffect(() => {
    const id = setTimeout(() => {
      console.log("Hello World", ref.current);
    }, 1000);
    return () => {
      clearTimeout(id);
    };
  }, [count]);

  const onClick = () => {
    setCount(count + 1);
    ref.current = count;
  };

  return <button onClick={onClick}>{children}</button>;
}
```

### useRef by Callback

但這樣要管理兩個狀態實在頗麻煩，所以你可能就需要使用另一個 useEffect 來解決。

```js
function App() {
  const [count, setCount] = useState(0);

  const callback = useCallback(() => {
    console.log("Hello World", count);
  }, [count]);

  const ref = useRef(callback);

  useEffect(() => {
    ref.current = callback;
  }, [callback]);

  useEffect(() => {
    const id = setTimeout(() => {
      ref.current();
    }, 1000);
    return () => {
      clearTimeout(id);
    };
  }, []);
}
```

從這例子來看，真正要被更新的不是 useEffect 的 callback，而是 `setTimeout` 的 callback。我重新建立一個 `useRef` 來儲存 `setTimeout` 的 callback，再透過 `useCallback` 來更新它的依賴狀態，這樣當 `count` 改變時，`setTimeout` 的 callback 也會被更新，這樣就不會有問題了。

### use EventTarget

但你也可能會說，這樣寫起來也頗麻煩的，而且 react hook 使用太多也頗浪費效能，也難以閱讀理解程式的邏輯。那是必要找出更精簡的方案。

```js
const EventType = {
  TIMEOUT: "timeout",
};

function App() {
  const [count, setCount] = useState(0);
  const ref = useRef(new EventTarget());

  useEffect(() => {
    const emitter = ref.current;
    const onTimeout = () => {
      console.log("Hello World", count);
    };
    emitter.addEventListener(EventType.TIMEOUT, onTimeout);
    return () => {
      emitter.removeEventListener(EventType.TIMEOUT, onTimeout);
    };
  }, [count]);

  useEffect(() => {
    const emitter = ref.current;
    const id = setTimeout(() => {
      emitter.dispatchEvent(new Event(EventType.TIMEOUT));
    }, 1000);
    return () => {
      clearTimeout(id);
    };
  }, []);
}
```

這裡我使用到了 `EventTarget` ，這是利用 Event Bus 的概念來解決這個問題，這樣就不需要使用到 `useRef` 和 `useCallback` 來解決，減少了 Hook 的運作次數。因為 Javascript 本來就是 Event Driven，所有的 Callback 本質就是 Event，我只是更顯示去用 Event 來解決非同步的事件。但這仍然有缺點，就是 `EventTarget` 是瀏覽器提供的 API，所以這樣寫起來會有跨平台的問題。

### use EventEmitter

當然，這種小問題並不難解決，我們可以站在巨人的肩膀上，使用 `EventEmitter` 來解決這個問題。只要安裝 `events` 套件，就可以解決這個問題。

```js
import { EventEmitter } from "events";

const EventType = {
  TIMEOUT: "timeout",
};

function App() {
  const ref = useRef(new EventEmitter());
  const [count, setCount] = useState(0);

  useEffect(() => {
    const emitter = ref.current;
    const onTimeout = () => {
      console.log("Hello World", count);
    };
    emitter.on(EventType.TIMEOUT, onTimeout);
    return () => {
      emitter.off(EventType.TIMEOUT, onTimeout);
    };
  }, [count]);

  useEffect(() => {
    const emitter = ref.current;
    const id = setTimeout(() => {
      emitter.emit(EventType.TIMEOUT);
    }, 1000);
    return () => {
      clearTimeout(id);
    };
  }, []);
}
```

這也太複雜了吧！確實越寫越複雜，但這才是合理疏通整個 Functional Programming 的邏輯。我透過事件來根本解決這問題，我定義了 setTimeout callback 事件，也建立一個這 callback 屬於自己的 dependency array，每一個 callback 都會有自己的 dependency array，這樣就不會有狀態同步問題，兩個 function 的依賴徹底被拆離。

## use EventEmitter with Promise callback

所以今天例子換成 Promise 也是一樣的道理，也同時背後可以解決 Strict Mode 產生的的問題。

```js
const EventType = {
  TIMEOUT: "timeout",
};

const sleep = (ms) => {
  return new Promise((resolve) => {
    setTimeout(resolve, ms);
  });
};

function App() {
  const ref = useRef(new EventEmitter());
  const [count, setCount] = useState(0);

  useEffect(() => {
    const onTimeout = () => {
      console.log("Hello World", count);
    };
    emitter.on(EventType.TIMEOUT, onTimeout);
    return () => {
      emitter.off(EventType.TIMEOUT, onTimeout);
    };
  }, [count]);

  useEffect(() => {
    const emitter = ref.current;
    let ignore = false;
    const id = sleep(1000).then(() => {
      emitter.emit(EventType.TIMEOUT);
    });
  }, []);
}
```

### Last Solution

那我們最後來一個輕裝封裝，來讓它看起來更舒服。

```js
const createEventBus = (event) => {
  let listeners = [];
  return {
    on: (callback) => {
      listeners.push(callback);
      return () => {
        listeners = listeners.filter((listener) => listener !== callback);
      };
    },
    emit: (...args) => {
      listeners.forEach((listener) => listener(...args));
    },
  };
};

function onTimeout(callback) {
  const id = setTimeout(() => {
    callback();
  }, 1000);
  return () => {
    clearTimeout(id);
  };
}

function App() {
  const timeoutRef = useRef(createEventBus("timeout"));
  const [count, setCount] = useState(0);

  useEffect(() => {
    return timeoutRef.current.on(() => {
      console.log("Hello World", count);
    });
  }, [count]);

  useEffect(() => {
    return onTimeout(() => {
      timeoutRef.current.emit();
    });
  }, []);
}
```

經過多次封裝後，它整個邏輯變得非常清晰，而且也更容易理解。同時解決上述全部問題。

## React Callback 的問題

你是否曾經呼叫了 Callback 後，卻發現它 State 並沒有更新？

```js
import sleep from "./sleep";

function App() {
  const [count, setCount] = useState(0);

  const onSleep = () => {
    console.log("onInterval before sleep", count);
  };

  const onInterval = () => {
    console.log("onInterval", count);
    sleep.then(onSleep);
  };

  const onClick = () => {
    setCount(count + 1);
  };

  return (
    <div>
      <Timer timeout={1000} onInterval={onInterval} />
      <button onClick={onClick}>{count}</button>
    </div>
  );
}
```

這案例，Timer 每 1000ms 會執行一次 `onInterval`，但 `onInterval` 內部又呼叫了 `sleep`，所以 `onSleep` 會在 `sleep` 執行完後才執行。你會發現 `onInterval` 的 `count` 是新的，但 `onSleep` 卻是與 `onInterval` 是相同的，並不是即時的。

但其實解決方案一樣，繼前面的案例手段，我們用一樣的方法可以去解決。

```js
import createEventBus from "./createEventBus";
import sleep from "./sleep";

function App() {
  const sleepRef = useRef(createEventBus("sleep"));
  const [count, setCount] = useState(0);

  useEffect(() => {
    return sleepRef.current.on(() => {
      console.log("onInterval before sleep", count);
    });
  }, [count]);

  const onSleep = () => {
    sleepRef.current.emit();
  };

  const onInterval = () => {
    console.log("onInterval", count);
    sleep.then(onSleep);
  };

  const onClick = () => {
    setCount(count + 1);
  };

  return (
    <div>
      <Timer timeout={1000} onInterval={onInterval} />
      <button onClick={onClick}>{count}</button>
    </div>
  );
}
```

多一個 `useEffect` 和 `useRef` 就精準解問題。

## 後記

可能原來寫 React 的人看完後會有很多不認同，但這就是我寫 React 的經驗。我寫 React 的時候，常常會遇到這些問題，所以我才會寫出這些解決方案。不同的方案有不同的 Coding Style，但最終目的都是為了讓程式碼更容易理解，更容易維護。
