# 前端也可以搞微服務？！前端最複雜的一種架構

## 使用 WebComponent 建立微前端

在使用微前端時，Web Component 常常被提及，他們雖然不是綁定的，但要徹底解耦脫鉤，它是很好用的手段。

### 建立一個 React 的微前端

前面說了那麼多，從這裡要實際來談談如何實現用 React + WebComponent 微前端。

#### 先決定方案

當你開始要導入微前端時，你就得先思考架構方案。

- 把 Component 做成 WebComponent，使用 customElements 註冊作為微前端
- 把一個 ReactComponent 做成微前端，直接載入 Component Module

#### 以 React 來製作 WebComponent 微前端

以 WebComponent 作為載體，重新建立一個 React Instance。

```jsx
import ReactDOM from "react-dom/client";
import App from "micro-component/app";

class ReactMicroElement extends HTMLElement {
  connectedCallback() {
    const mountPoint = document.createElement("div");
    this.appendChild(mountPoint);
    const root = ReactDOM.createRoot(mountPoint);
    root.render(<App />);
  }
}

customElements.define("react-app", MicroElement);
```

該方法延展性很好，隨時可以替換成不同框架的元件，也可以轉成 Shadow DOM 作隔離，也可以在 DOM 上加工做不同的使用。當如果在不同微前端使用的套件版本不一致時，是很容易升級跟並存的。

#### 以 ReactComponent 作為微前端

這個方案是最容易被實作，而且比較沒有溝通問題，可以直接使用 React Context 進行溝通。

```jsx
const MicroApp = lazy(() => import("micro-component/app"));
```

這種方法雖然非常受到歡迎，但地獄的地方也在這，他對於套件兼容能力很差，當你使用了某個套件時，他將會要求所有微前端的版本必須是一致相同，否則無法共享記憶體。如果是想要分散式部署時，這將是一場災難，因為你本身就很難管理多個微前端的套件應該是一致的，特別是又依賴於某個框架，那更是難以同步。

### 建立一個 Vue 的微前端

Vue 相對於說在建立微前端是容易的，大多時候只要處理好事件與元件間的溝通，元件與元件大多不依賴相同的 Vue 實體，對於生命週期處理與依賴注入處理相對寬鬆，也提供抽象化成 WebComponent 的方案，所以是相對容易的。

- 把 Component 做成 WebComponent，使用 Vue 提供的 defineCustomElement 註冊作為微前端
- 把一個 VueComponent 做成微前端，直接載入 Component Module

#### 以 Vue 來製作 WebComponent 微前端

以 WebComponent 作為載體，可以更容易進行跨應用組件溝通，比起傳統的手段會更加容易。

```js
const VueMicroElement = defineCustomElement({
  props: {},
  emits: {},
  template: `...`,
  styles: [`/* inlined css */`],
});

customElements.define("vue-app", VueMicroElement);
```

該方法延展性很好，隨時可以與各種框架共同協作，也可以在 DOM 上加工做不同的使用。當如果在不同的 Vue 與微前端使用的套件版本不一致時，是很容易升級跟並存的。

#### 以 VueComponent 作為微前端

以 VueComponent 作為載體，使用起來比較沒有溝通困難，但對於跨框架上還是需要做一些加工。這方法寫法十分簡單，只要打包好就很容易被使用，也沒有太多應用難點。但要特別注意內部的 `vue/reactivity` 是不會互相溝通的，響應能力會比較差，要特別小心資料變化通知要透過 VueComponent 的 event and props 來傳遞。

```js
const VueMicroElement = defineAsyncComponent(() =>
  import("micro-component/app")
);
```
