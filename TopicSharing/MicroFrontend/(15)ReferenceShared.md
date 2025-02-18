# 前端也可以搞微服務？！前端最複雜的一種架構

## Reference Shared

前面談了那麼多的觀念與理論，還是要來講點實務應用上的東西。

### 記憶體共享問題在哪？

我們過去在 Monolith 都覺得共用是程式碼優化最高原則，這就是所謂的「聚合性」。但過度追求聚合性的下場就是帶來高度耦合，在微前端這議題更是如此。微前端有一項很重要的議題，就是「共享記憶體」。

先看看下面這段：

```js
let data = null;

export function setData(value) {
  data = value;
}

export function getData() {
  return data;
}
```

很多套件喜歡設計類似於這樣的設計，這是一種閉包(Closure)技巧的延伸，非常好用。可以用來解耦共用，更彈性封裝了狀態管理。但這件事在跨應用溝通時不免成了問題。很多框架套件當初並不會認為你會載入很多份，所以把自己本身就當作是一個 store 暫存了一些狀態資訊。舉凡 i18n, react... 等等你喊的出來的知名套件都有這樣的設計在裏面，但模塊一但各自運行，記憶體位置是不一致的。

```js
function pluginA() {
  return class Plugin {};
}

function pluginB() {
  return class Plugin {};
}

const PluginA = pluginA();
const PluginB = pluginB();

const plugin = new PluginA();

console.log(plugin instanceof PluginB);
```

以這範例，我們雖然看到 pluginA,pluginB 在自己作用域有相同的名稱與實作，但實際上就是使用了不同的記憶體位置在工作。想當然，最後的 `console.log` 的答案是 false，但這其實就成為了微前端溝通的難題。所以才會依賴 ModuleFederation 去協助共享套件的 Reference。

### 共享就沒事了？

如果你認為就乾脆全部共享就沒事了？那還真是大錯特錯...因為你可能不想要全部共享。在共享記憶體時有一個很大的前提，就是程式原始碼必須完全一樣，講直白話就是要「所有想共享的 Package 版本必須一致相同」。想想發現什麼不對勁嗎？

仔細想想當初為什麼選擇微前端？就是為了模塊自治，讓每個模塊做到最少的耦合，獨自發布獨自部署獨自運行。然而模塊要溝通就變成惱人的問題，切也不是，不切也不是。不共享你無法溝通，很多套件功能會失效，導致運行異常。一但溝通又造成版本依賴，他們強迫被耦合在一起，萬一你想要任何套件升版就是「全部聯邦都得一起升版」。

這就回來講到，為什麼 HTTP 協定是「無狀態」，跨越微服務的界線就是溝通不能帶著狀態，否則無法自治。這件事在前端成了問題，甚至很嚴重，因為無法共享代表 bundle size 會大幅增加，對 Client 成了相當的壓力。

最後我想說的是，共享前再想想，真的有必要共用嗎？你要解耦還是縮小程式碼大小?想清楚策略的目的與結果。
