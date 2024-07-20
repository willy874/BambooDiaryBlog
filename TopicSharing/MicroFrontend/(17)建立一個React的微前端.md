# 前端也可以搞微服務？！前端最複雜的一種架構

## 建立一個 React 的微前端

前面說了那麼多，從這裡要實際來談談如何實現微前端。

### 先決定方案

當你開始要導入微前端時，你就得先思考架構方案。

- 把 Component 做成 WebComponent，使用 customElements 註冊作為微前端
- 把一個 ReactComponent 做成微前端，直接載入 Component Module

### 以 WebComponent 作為微前端

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

### 以 ReactComponent 作為微前端

這個方案是最容易被實作，而且比較沒有溝通問題，可以直接使用 React Context 進行溝通。

```jsx
const MicroApp = lazy(() => import("micro-component/app"));
```

這種方法雖然非常受到歡迎，但地獄的地方也在這，他對於套件兼容能力很差，當你使用了某個套件時，他將會要求所有微前端的版本必須是一致相同，否則無法共享記憶體。如果是想要分散式部署時，這將是一場災難，因為你本身就很難管理多個微前端的套件應該是一致的，特別是又依賴於某個框架，那更是難以同步。

### 我該採用哪個？

當你業務邏輯相對隔離，是屬於一個由不同團隊同時維護的大型專案時，我推薦你採用「以 WebComponent 作為微前端」這個方案，它可以大大降低不同團隊風格差異帶來的影響。

但當你團隊小，一個團隊需要碰到多塊不同功能的微前端，那麼「以 ReactComponent 作為微前端」會更適合，實作容易，傳遞溝通容易，避免建立一堆溝通機制。
