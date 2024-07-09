# 前端也可以搞微服務？！前端最複雜的一種架構

## 建立一個 Vue 的微前端

Vue 相對於說在建立微前端是容易的，大多時候只要處理好事件與元件間的溝通，元件與元件大多不依賴相同的 Vue 實體，對於生命週期處理與依賴注入處理相對寬鬆，也提供抽象化成 WebComponent 的方案，所以是相對容易的。

- 把 Component 做成 WebComponent，使用 Vue 提供的 defineCustomElement 註冊作為微前端
- 把一個 VueComponent 做成微前端，直接載入 Component Module

### 以 WebComponent 作為微前端

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

### 以 VueComponent 作為微前端

以 VueComponent 作為載體，使用起來比較沒有溝通困難，但對於跨框架上還是需要做一些加工。這方法寫法十分簡單，只要打包好就很容易被使用，也沒有太多應用難點。

```js
const VueMicroElement = defineAsyncComponent(() =>
  import("micro-component/app")
);
```

### 我該採用哪個？

在 Vue 上這並沒有太多議題，你只要確保你的響應溝通渠道是仰賴 Component 的 props 和 emit，框架會幫你完成兩邊的連動。採用 WebComponent 當然多少會面臨 Bundle Size 比較肥大的問題，這時只要做好控管，都不會有太大問題。比較咬慎重的是，採用 VueComponent 會比較方便，採用 WebComponent 可以容易跨框架，但也得謹慎去評估是否共用相關套件，以這樣的情況去評估。
