# 前端也可以搞微服務？！前端最複雜的一種架構

## WebComponent 的微前端溝通

WebComponent 不只可以脫離框架建立組件，還可以做多種形式的溝通。

### Attribute

WebComponent 最主要的溝通橋樑就是 Attribute，可以任意定義非原生的 Attribute 來進行資料傳遞。

但我會特別推薦使用 dataset 的屬性，如 `data-title`，而不是直接使用 `title`，因為 `title` 是原生 HTML 屬性，很容易跟許多原生功能有衝突。

```html
<custom-app data-title="I am title"></custom-app>
```

而 WebComponent 內部該怎麼做呢？如果查詢 MDN 會有 `attributeChangedCallback` 這個 event hook，但事實上是接不到任何修改資訊，你還得搭配 `observedAttributes` 才能觸發響應。

```js
class CustomComponentElement extends HTMLElement {
  observedAttributes = ["data-title"];

  attributeChangedCallback(attributeName, oldValue, newValue) {
    if (attributeName === "data-title") {
      // impl...
    }
  }
}

customElements.define("custom-component", CustomComponentElement);
```

這種方式坦白講很麻煩，所以我後來實作另一種方式。我改用 `MutationObserver` 來監聽，這樣就不需要事前監聽指定屬性，然後再掛勾回 dom 去用 event 的方式監聽。

```js
class AttributeChangedEvent extends Event {
  constructor({ attributeName, oldValue, newValue }) {
    super("attributechanged");
    this.attributeName = attributeName;
    this.oldValue = oldValue;
    this.newValue = newValue;
  }
}

class CustomComponentElement extends HTMLElement {
  #observer: MutationObserver;
  constructor() {
    super();
    this.#observer = new MutationObserver((mutationList) => {
      for (const mutation of mutationList) {
        if (
          mutation.target === this &&
          mutation.type === "attributes" &&
          typeof mutation.attributeName === "string"
        ) {
          this.dispatchEvent(
            new AttributeChangedEvent({
              attributeName: mutation.attributeName,
              oldValue: mutation.oldValue,
              newValue: this.getAttribute(mutation.attributeName),
            })
          );
        }
      }
    });
    this.#observer.observe(this, { attributes: true, attributeOldValue: true });
  }
}

customElements.define("custom-component", CustomComponentElement);

document
  .querySelector("custom-component")
  .addEventListener("attributechanged", (event) => {
    // impl...
  });
```

### Event

事件是在微前端中最方便的一種溝通方式，它依賴 DOM 的樹狀溝通渠道，但卻能有效達到下對上的事件溝通。

```js
const el = document.querySelector("custom-component");

el.addEventListener("customevent", (event) => {});
```

```js
const child = document.querySelector("custom-component-child");

child.dispatchEvent(new Event("customevent"));
```

### Slot

插槽可以更彈性去穿插並調整 WebComponent 的展示層級，也相等於傳遞 Component 的資訊給下層。

```html
<custom-component>
  <span slot="custom-text">slot text!</span>
</custom-component>
```

```js
class CustomComponentElement extends HTMLElement {
  constructor() {
    super();
    this.attachShadow({ mode: "open" });
    this.shadowRoot.innerHTML = `
    <style>
      span {
        color: #333;
      }
    </style>
    <p>
      <slot name="custom-text">
        <span>custom default text</span>
      </slot>
    </p>
  `;
  }
}
customElements.define("custom-component", CustomComponentElement);
```

### Resource

- [Using custom elements](https://developer.mozilla.org/en-US/docs/Web/API/Web_components/Using_custom_elements)
- [MutationObserver](https://developer.mozilla.org/en-US/docs/Web/API/MutationObserver)
