### 背景

dom操作是js编程最重要的一部分，所以对于前端开发来说，需要深入掌握它的一些知识。

### Document

在Javascript mdn上有对`document`的解释说明:

```
Document 接口提供了一些在浏览器服务中作为页面内容入口点而加载的一些页面，也就是 DOM 树。 DOM 树包括诸如 <body> 和 <table> 之类的元素，及其他元素。其也为文档（document）提供了全局性的函数，例如获取页面的 URL、在文档中创建新的 element 的函数。
Document接口描述了任何类型的文档的公共属性和方法。根据文档的类型 (例如 HTML、XML、SVG,  ... ), 可以使用更大的API： HTML 文档, 以text / html内容类型提供, 也实现了HTMLDocument接口，而SVG 文档实现了 SVGDocument 接口。
```

在`lib.es6.d.ts`中有对Document接口的定义

```typescript
interface Document extends Node, GlobalEventHandlers, NodeSelector, DocumentEvent, ParentNode, DocumentOrShadowRoot {
  ...
}
```

这里可以看出`Document`继承了很多接口，在浏览器`console`输入`Document.prototype`可以查看`Document`的完整继承关系，如下图：

![Document](http://oznz1caxs.bkt.clouddn.com/Document.png)

下面我会来介绍一下Document中常用的属性和方法，属性中还有一些只读的属性，这些需要关注下他们的具体意义，例如:

```typescript
interface Document {
  ...
  /**
     * Gets an object representing the document type declaration associated with the current document.
     */
  readonly doctype:DocumentType;
  ...
}
```

一般我们操作的`document`并不是上面所说的`Document`类，而是`HTMLDocument`接口生成，`HTMLDocument`继承于`Document`，主要重载了`EventTarget` 中的`addEventListener`方法，具体定义如下：

```typescript
interface HTMLDocument extends Document {
    addEventListener<K extends keyof DocumentEventMap>(type: K, listener: (this: HTMLDocument, ev: DocumentEventMap[K]) => any, useCapture?: boolean): void;
    addEventListener(type: string, listener: EventListenerOrEventListenerObject, useCapture?: boolean): void;
}
```

`HTMLDocument`继承于`Document`接口，`Document`中有一些常用的恩方法和属性，`Document`是一个接口，我们可以通过`new`来生成一个`Document`对象:

```javascript
// 这不是标准的特性，所以生产环境尽量避免使用
var doc = new Document();
```

 

### document

页面上经常操作的`document`对象继承了`Document`接口，下面来介绍一些常用的：

```javascript
// 获取页面上获得焦点的html元素
// 这是一个只读的属性，返回一个实现Element接口的对象
// readonly activeElement: Element;
document.activeElement 
```

```javascript
// 返回当前文档中的<body>元素或者<frameset>元素
// body: HTMLElement;
document.body
```

```javascript
// 返回cookie字符串
document.cookie
```

除了一些重用的属性，`Document`中还有一些常用的dom事件，例如：

```typescript
interface Document {
  /**
    * Fires when the user aborts the download.
    * @param ev The event.
    */
  onabort: (this: Document, ev: UIEvent) => any;
  /**
    * Fires when the object loses the input focus.
    * @param ev The focus event.
    */
  onblur: (this: Document, ev: FocusEvent) => any;
  /**
    * Fires when the user clicks the left mouse button on the object
    * @param ev The mouse event.
    */
  onclick: (this: Document, ev: MouseEvent) => any;
}
```

还有一些`dom`操作的方法，例如`createElement`、`createTextNode`、`getElementById`等。这些都是我们日常书写中常用的方法，那节点操作的方法和属性又在哪个接口里面呢，注意一下我们`Node`接口，这个接口里面定义了上述的节点操作，例如`appendChild`、`hasAttributes`、`insertBefore`，看下`Node`接口中的一些常用的：

```typescript
interface Node extends EventTarget {
  readonly attributes: NamedNodeMap;
  readonly baseURI: string | null;
  readonly childNodes: NodeList;
  readonly firstChild: Node | null;
  readonly lastChild: Node | null;
  appendChild<T extends Node>(newChild: T): T;
  cloneNode(deep?: boolean): Node;
  compareDocumentPosition(other: Node): number;
  contains(child: Node): boolean;
  hasAttributes(): boolean;
  hasChildNodes(): boolean;
  insertBefore<T extends Node>(newChild: T, refChild: Node | null): T;
}
```

那么一些选择dom节点新出的api在哪个接口呢，例如`querySelector`和`querySelectorAll`，这两个方法我们可以在`NodeSelector`接口中找到，并且是`Document`是继承他的:

```typescript
interface Document extends NodeSelector {}
```

```typescript
interface NodeSelector {
  querySelector<K extends keyof ElementTagNameMap>(selectors: K): ElementTagNameMap[K] | null;
  querySelector(selectors: string): Element | null;
  querySelectorAll<K extends keyof ElementListTagNameMap>(selectors: K): ElementListTagNameMap[K];
  querySelectorAll(selectors: string): NodeListOf<Element>;
}
```

`Document`也继承了`ParentNode`接口，能够获取到直接子元素，接口定义如下：

```typescript
interface ParentNode {
  readonly children: HTMLCollection;
  readonly firstElementChild: Element | null;
  readonly lastElementChild: Element | null;
  readonly childElementCount: number;
}
```

除了上述一些接口之外，还有一个接口里面的方法可能会用到，`DocumentOrShadowRoot`接口，他能够获取到页面上`stylesheet`集合，或者根据坐标定位html元素，接口具体定义如下：

```typescript
interface DocumentOrShadowRoot {
    readonly activeElement: Element | null;
    readonly stylesheets: StyleSheetList;
    getSelection(): Selection | null;
    elementFromPoint(x: number, y: number): Element | null;
    elementsFromPoint(x: number, y: number): Element[];
}
```

了解完上述的接口之后，可以看下整个继承链最后一个接口`EventTarget`，下面看下接口的具体定义：

```typescript
interface EventTarget {
    addEventListener(type: string, listener?: EventListenerOrEventListenerObject, options?: boolean | AddEventListenerOptions): void;
    dispatchEvent(evt: Event): boolean;
    removeEventListener(type: string, listener?: EventListenerOrEventListenerObject, options?: boolean | EventListenerOptions): void;
}
```

这个接口定义最基础的事件添加和出发，包括三个方法`addEventListener`、`dispatchEvent`、`removeEventListener`，下面是一个最基础的事件绑定触发的例子:

```javascript
const ev = new Event('build')
document.addEventListener('build', (e) => {console.log('dispatch')}, false)
document.dispatch(ev) // "dispatch"
```



### HTMLElement

通过`document.querySelector`获得的dom都是继承`HTMLElement`接口的，这个接口定义了元素的一些基础属性如`offsetHeight`、`innerText`等，还定义了一些基础dom事件，下图是`HTMLElement`的继承关系：

![HTMLElement](http://o9kn1bu4x.bkt.clouddn.com/HTMLElement.png)

`Element`接口也定义一些属性和方法，包括最近常用的`classList`。

上述所有的接口的属性和方法均可以在https://developer.mozilla.org查询到。