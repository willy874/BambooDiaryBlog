# 前端也可以搞微服務？！前端最複雜的一種架構

## ModuleFederation-RemoteChunk

前面講那麼多關於微前端的相關技術，但說到微前端的打包前溝通就不得不談談 ModuleFederation，這篇開始就要隆重介紹 ModuleFederation！

### 什麼是 ModuleFederation?

模塊聯邦（Module Federation，以下均使用 MF 簡稱），它是一種使用 JavaScript 分離管理模塊的一種架構方式，在每個資源位置放置了一份自己的完整資源，依照進入點的設計，它像其他資源請求時，會載入遠端資源中自己依賴缺失的相關資源，也會排除載入自己擁有的資源，來達到模塊共享與記憶體共享。

### 所以 ModuleFederation 怎麼工作？

講定義很撓口，我們來談談他怎麼幫你建構這個聯邦。

### Resource

- [Module Federation](module-federation.io)
- [Webpack Module Federation](https://webpack.js.org/concepts/module-federation/)
