# 前端也可以搞微服務？！前端最複雜的一種架構

## ModuleFederation

前面講那麼多關於微前端的相關技術，但說到微前端的打包前溝通就不得不談談 ModuleFederation，這篇開始就要隆重介紹 ModuleFederation！

### 什麼是 ModuleFederation?

模塊聯邦（Module Federation，以下均使用 MF 簡稱），它是一種使用 JavaScript 分離管理模塊的一種架構方式，在每個資源位置放置了一份自己的完整資源，依照進入點的設計，它像其他資源請求時，會載入遠端資源中自己依賴缺失的相關資源，也會排除載入自己擁有的資源，來達到模塊共享與記憶體共享。

### 所以 ModuleFederation 怎麼工作？

講定義很撓口，我們來談談他怎麼幫你建構這個聯邦。

當你在執行打包時，其實就是把所有同步依賴的模組捆綁成一大包 JS 檔案，當你去索求這些依賴的時候其實都已經加載好了。但這一大包的檔案，如果分散式部署在許多伺服器資源就很浪費，會有相當多重複的程式碼。此時可以把一些固定的資源轉寫成 Map，以 package and version 兩個判斷點作為 key，這樣就可以建立模組化的加載點，也可以切分作用域以防衝突。但總不能在應用初始化一開始就把整個應用程式全部的套件都載完吧？所以會在所有可能用到的遠端資源上放對應的可能用到的套件模組，等到需要時再加入這個 Map，這樣就形成了一個一塊自治的運作區塊，就算沒有其他功能支援也完全能獨立運作。一但遇到重複套件就會去加載已經曾經讀取過的套件，不再去讀取自己那份，達到共用的目的。

所有聯邦模塊都能夠自制運行，也可以協同運作，依照業務場景與使用者旅程相互支援，這就是 ModuleFederation(模塊聯邦)。

### 所以，跟微前端什麼關係？

MF 並不只是解決共用問題，他同時做到功能拆分、作用域分隔、版本控管等等，解決許多套件管理上許多問題，後面會來說說 MF 倒底用什麼規則來管理共享這議題。很多人提到微前端，反射神經就會想到 MF。但 MF 並不是微前端「必要的」存在，甚至並不是一個 Framework ，它只是一種 bundle 的概念，也只是一個共享程式和共享記憶體的一種手段。

MF 是把各個套件的各個版本分散式存在每一個微服務端點，透過 entry point 的 map 確認是否曾經加載過，當曾經加載過特定版本的 module 之後，就不會重複加載，並且載入同一包被快取的 JavaScript 也可以共享記憶體。而這個機制便是 Webpack5 提供的一個打包機制，也是在微前端去解決「版本管理」、「模塊共享」、「記憶體共享」的關鍵機制，這也是為什麼很多人會把這兩個東西劃上等號。反過來說，其實不使用 MF 也是能夠處理這樣的機制，但這就牽扯微前端的溝通框架，能否進行這樣的溝通和封裝。通常還是蠻仰賴 Bundle Tool 在預處理，否則處理起來實在是相當囉嗦。

模組共用是微前端在切分時，跟微服務最大的不同。前端是有流量上的消費，每一個客戶端都會造成伺服器一定的負擔，所以像是微服務單純的切分，微前端更要經常煩惱「共用」議題。這又不得不強調，MF 並不是一個套件，而是一種多模組的管理系統的思維，只是有套件實現了這種思維。

MF 的理念基本上貫穿所有「模組共用」的思維，使用多服務進入點管理多版本的 packages，具備版本識別與切分環境的能力，讓多個組態情境能良好管理各自的環境版本。目前其實幾乎找不到比起 MF 更完善的其他解決方案，幾乎仰賴大量的基礎建設架構的實作實現，我個人還是建議就用吧。一旦使用了 MF，模組化就會被綁定在 特定的 chunk system ，遷移上變成整個架構的成本。微前端雖然有分散部署的優點，但架構面的部分仰賴一致的規範與協定，如果一致的部分發生變動，那就會根本性的需要大幅度調整。

### 除了 Webpack 外，那 Vite 呢？

Vite 的 Module Federation 算是 Webpack 的閹割版，也不支援 Vite 的 dev mode，所以使用上開發體驗沒有想像中好。我的經驗上我還是以 Webpack 為主。如果你的微前端情境足夠複雜，我暫時來說依然還是建議你採用 Webpack Module Federation。但如果還不到太複雜的設計與應用，其實開發可以利用 `build + preview` 的模式來達到 MF 共享，但仍然沒有原本的開發體驗好。截至本文撰稿日，我仍然在等 Vite 官方去提供更加完善的 MF 核心方案，希望能根本性解決 Vite 在使用 MF 的 DX 不良問題。

### Resource

- [Module Federation](module-federation.io)
- [Webpack Module Federation](https://webpack.js.org/concepts/module-federation/)
- [Vite Module Federation](https://github.com/originjs/vite-plugin-federation)
- [一文通透讲解 webpack5 module federation](https://juejin.cn/post/7048125682861703181)
- [Understanding Module Federation: A Deep Dive](https://scriptedalchemy.medium.com/understanding-webpack-module-federation-a-deep-dive-efe5c55bf366)
