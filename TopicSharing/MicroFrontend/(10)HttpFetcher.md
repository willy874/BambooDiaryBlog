# 前端也可以搞微服務？！前端最複雜的一種架構

## 全域單例狀態管理-HttpFetcher

### 什麼是 HttpFetcher？

先說明 HttpFetcher 是什麼。當前主流的 HttpFetcher 應該就是 axios, fetch ... 等等方式，凡是向網路發出請求的方法都是 HttpFetcher 相關的工具。

### 有什麼問題？

大部分我們在處理 Http Request 時，都會有一些「固定要做的事」。精準來說要舉例就是一些共同的處理，比如說將 Token 注入、請求的錯誤處理、回應的錯誤處理、回應的處理等等，相關的中介層行為肯定不會想重複編寫，大部分行為對於一個網站也幾乎是一致的，不太會有不同頁有不同的處理。所以在微前端的架構如何去共用這個行為呢？

### 解決方案

#### 建立 Client Service

既然是 HTTP 中介層，使用的處理行為就未必要在瀏覽器進行完成，你可以選擇在伺服器進行請求處理，如 Token 管理就可以在伺服器端完成。

#### 連段式 Event

透過層層事件把行為透過 EventBus 傳遞給每一層事件處理，算是另類的 middleware，有點 `Chain of Responsibility Pattern` 的感覺。除了可以用 deep 的方式去深層傳遞，也可以用 `Next Event` 的手法去幫助層層傳遞。EventBus 更方便讓微前端之間進行溝通與攔截，達到更高的自由度。

#### 建立共用 Instance

這方式就是建立一個抽象實體去讓全部請求共用，管理這一份記憶體也可以很好控制所有 Interceptor 行為。

#### Interceptor Service

強行攔截所有 HTTP 請求，重新封裝 XHR 與 Fetch 行為，在 ServiceWork 重新處理請求行為。

#### 不考慮共用

直接放棄所有共用行為，每一個微前端應用程式都獨立執行對應的行為。
