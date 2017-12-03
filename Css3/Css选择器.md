### CSS Specificity（CSS特异性）

书写css的时候，需要知道选择器的优先级。 避免在为同一个element写样式的时，前面的样式覆盖了后面的样式。

所以需要计算一下我们书写的选择器的优先级，那应该如何计算呢？看一下下面的规则：

1. `*`号，在任何情况下记为0
2. 类型选择器例如`div`、`h2`和伪类选择器例如`:first-child`，计数1
3. 属性选择器例如例如`[link="rel"]`和class选择器例如`.gbb`，计数10
4. id选择器例如`#app`，计数100

如下是一些例子

| 选择器                | 计数                                  |
| ------------------ | ----------------------------------- |
| * {}               | 0                                   |
| li {}              | 1 (one element)                     |
| li:first-line { }  | 2 (one element, one pseudo-element) |
| ul li { }          | 2 (two elements)                    |
| h1 + *[rel=up] { } | 11 (one attribute, one element)     |
| ul ol li.red { }   | 13 (one class, three elements)      |
| li.red.level { }   | 21 (two classes, one element)       |
| #sith              | 100 (one id selector)               |



### 属性选择器(Attribute selectors)

属性选择器根据属性去定位一个元素，通过指定元素的属性，html中具有属性的元素都会称为目标。常用的属性选择器有如下7种：

1. `[att]` 通过属性的名称定位元素

   ```html
   <div name="gbb">gbb</div>
   <div name="acy" sexy="female">acy</div>
   ```

   ```css
   div[name] { color: pink } /* 作用于第一个和第二个元素 */
   div[name][sexy] { width: 150px } /* 作用于第二个元素 */
   ```

2. `[att=value]` 该属性匹配确定的value

3. `[att~=value]` 该属性的值需要以空格分隔的单词列表（例如class =“title featured home”），其中一个单词恰好是指定的值。

   ```html
   <div class="title">gbb</div>
   <div class="titlefeaturedhome">acy</div>
   <div class="title featured home">wjw</div>
   ```

   ```css
   [class~="title"] { color: pink } /* 作用于第一个和第三个元素 */
   ```

4. `[att|=value]`  该属性能够匹配两种情况，精确匹配value，或者是以value开头后面跟一个`-`

   ```html
   <div name="gbb">gbb</div>
   <div name="gbb-">gbb-</div>
   ```

   ```css
   [name|="gbb"] { color: pink } /* 作用于第一个和第二个元素 */
   ```

5. `[att^=value]` 该属性匹配以value开头的元素

   ```css
   input[name^="gbb"] { width: 150px } 
   ```

6. `[att&=value]` 该属性匹配以value结尾的元素

   ```css
   input[name^="acy"] { width: 150px } 
   ```

7. `[att*=value]` 该属性匹配包含value的元素

   ```css
   input[name*="bbac"] { width: 150px }
   ```



###  子类选择器(Child Selectors)

子选择器由符号“>”表示， 它可以定位一个特定元素的直接子元素。例如：

```html
<div>
	<p> 
      	this is first level
  		<p> this is second level </p>
  	</p>
</div>
```

```css
span { color: black }
div > span { color: pink } /* this is first level 变色 */
```



### 兄弟选择器(Sibling Selectors)

1. (邻接兄弟选择器)此选择器使用加号“+”来组合两个简单选择器序列。 选择器中的元素具有相同的父元素，第二个元素必须在第一个之后立即出现。

   ```html
   <div>
   	<div>tell me name:</div>
     	<p>wjw</p>
       <p>gbb</p>
     	<p>acy</p>
   </div>
   ```

   ```css
   div + p { color: pink }  /* wjw 将会变色 */
   ```

2. (通用兄弟选择器)此选择器使用加号“~”来组合两个简单选择器序列。选择器中的元素具有相同的父元素，第一个元素之后的元素都可以选中，并且不需要相邻。

   ```html
   <div>
   	<div>tell me name:</div>
     	<h2>not</h2>
     	<p>wjw</p>
     	<h2>accaca</h2>
       <p>gbb</p>
     	<p>acy</p>
   </div>
   ```

   ```css
   div ~ p { color: pink } /* 所有p都会变色 */
   ```

   ​

### 伪类选择器(Pseudo-classes Selectors)

伪类总是以冒号`:`作为前缀，接着跟上伪类的名称，如`:link`、`:hover`、`:first-child`、`:nth-child`。(`::`双冒号属于伪元素前缀，实际不需要在html文档中书写，通过css创建的element)

##### :link :visited :hover :active

- `:link `作用于正常状态未访问过的a标签
- `:visited` 作用于访问过的a标签
- `:hover` 在鼠标滑动浮动在元素之上时会起作用
- `:active` 在鼠标或者键盘click时会起作用，访问过的将不会起作用



##### :focus

`:focus`伪类用于设备中元素获取焦点时，经常用于表单元素中

```css
input:focus { background: #eee }
```



##### :first-child :last-child

`:first-child` 伪类作用于父元素的第一个子元素

`:last-child`伪类作用于父元素的最后一个子元素

```html
<div>
  <p> gbb </p>
  <p> wjw </p>
  <p> acy </p>
</div>
```

```css
p:first-child{ color: pink } /* 第一个p元素会变色 */
p:last-child{ color: pink } /* 最后一个p元素会变色 */
```



##### :first-of-type :last-of-type

`:first-of-type`伪类标识父元素同类元素的第一个，和`:first-child`区别在于不需要一定在所有元素中的第一个。

`:last-of-type`伪类标识父元素同类元素的最后一个

```html
<div>
	<h2>title 1</h2>
  	<p>mike</p>
  	<h2>title 2</h2>
  	<p>tony</p>
</div>
```

```css
h2:first-of-type, p:first-of-type{ color: pink } /* 同类元素第一个 */
h2:last-of-type, p:last-of-type{ color: pink } /* 同类元素最后一个 */
```



##### :not

`:not`伪类接收选择器或者是伪类，匹配到没有被接收器包含的元素，并且可以写成链式操作。

```html
<div>
  <p class="w-text">part one</p>
  <p>part two</p>
  <p>part three</p>
</div>
```

```css
p:not(.w-text):not(:last-child) { color: pink } /* 中间第二个会变色 */
```



##### :nth-child :nth-last-child

`:nth-child`伪类根据传入的参数作用于一个活多个元素，具体有以下几种形式

- `li:nth-child(1)`作用于子元素的第一个元素
- odd,even参数分别作用于奇数元素和偶数元素
- 或者接受一个带有n的参数表达式，n为自然数

`:nth-last-child`类似于`:nth-child`，后者从头部开始计数，后者从尾部开始计数

```html
<ul>
  <li>ynd1</li>
  <li>ynd2</li>
  <li>ynd3</li>
  <li>ynd4</li>
  <li>ynd5</li>
  <li>ynd6</li>
</ul>
```

```css
li:nth-child(1){ color: pink }/* 作用于第一个li元素 */
li:nth-last-child(1){ color: orange } /* 作用于最后一个li元素 */
```

```css
/* 所用子元素都会变色 */
li:nth-child(even){ color: pink }
li:nth-last-child(even){ color: orange }
```

```css
li:nth-child(2n+5){ color: pink } /* 倒数第二个会变色 */
li:nth-last-child(2n+5){ color: orange } /* 正数第二个会变色 */
```



##### :nth-of-type :nth-last-of-type

`:nth-of-type`和`nth-child`类似，不通在于计数前者是同元素计数，而后者则是所有子元素一起计数。

```html
<div>
  <p>part 1</p>
  <div>level 1</div>
  <p>part 2</p>
  <div>level 2</div>
  <div>level 3</div>
  <p>part 3</p>
</div>
```

```css
p:nth-of-type(2){ color: pink } /* 第二个p元素会起作用 */
div:nth-of-type(3){ color: orange } /* 第三个div元素会起作用 */
```



##### :olny-child :only-of-type

`:only-child`会作用于父元素中只有一个子元素的情况

`:only-of-type`会作用于父元素中只有一个同类子元素的情况

```html
<div>
  <p>part 1<p>
<div>
<div>
  <h2>title 1</h2>
  <p>part 2</p>
</div>
```

```css
p:only-child{ color: pink } /* 作用于第一个div中的p */
p:only-of-type{ color: orange } /* 作用于第二个div中的p */ 
```



##### :target

`:target`  将元素的唯一标识和url的`#`做为目标

例如一个链接为 `http://www.testdemo.com#newpost`

```html
<div id="newpost">
	<h2>target</h2>	
</div>
```

```css
#newpost:target {
  width: 100%;
  height: 200px;
  color: pink
}
```



##### :checked

`:checked`伪类在form表单中例如radio或者checkbox被选中会起作用

```html
<form>
  <input type="checkbox" />
  <label>选中会变色</label>
</form>
```

```css
input:checked + label { color: pink }
```

##### :default

`:default`伪类在form表单中默认选中的会起到作用，但是当变为不选中状态时，状态依然存在，所以这个属性一般不适用

```html
<input type="checkbox" checked />
```

```css
input:default{box-shadow: 0 0 2px 1px coral;}
```



##### :disabled :enabled

`:disabled`伪类作用于元素disabled时，解除diabled状态时，将不会再起作用

`:enabled`伪类与disabled相反。

```html
<input type="text" value="gbbacy" disabled />
<input type="text" value="neverthanmore" />
```

```css
input:disabled{ color: pink }
input:enabled{ opacity: 0.5 }
```



##### :empty

`:empty`伪类作用于元素内不含有任何内容的情况，哪怕是一个空格。

```html
<div>gbbacy</div>
<div> </div>
<div>neverthanmore</div>
<div></div>
```

```css
div {
  background: orange;
  height: 30px;
  width: 200px;
  margin-bottom: 10px
}

div:empty {
  background: yellow;  /* 最后一个div背景颜色会变 */
}
```



##### :in-range :out-of-range

这两个伪类分别在form表单中范围之内或者之外会起作用例如number

```html
<input type="number" min="5" max="20" />
```

```css
input:in-range{border: 1px solid #eee}
input:out-of-range{border: 1px solid red}
```



##### :indeterminate

`:indeterminate`伪类作用于选择框不确定的状态，可以用js来操作。

```html
<input type="checkbox" id="check" />
<label>indeterminate状态</label>
```

```js
document.querySelector("#check").indeterminate = true
```

```css
input:indeterminate + label{
   	color: red
}
```



##### :valid :invalid

`:valid`伪类作用于form表单中格式正确的元素，例如email表单

`:invalid` 与`:valid`相反

```html
<input type="email" id="email" />
```

```css
input:valid{ color: pink }
```



##### :optional

只要输入不具有必需的属性，`:optional`就可以使用

```html
<input type="text" />  
```

```css
:optional{color: pink}
```



##### :read-only :read-write

`:read-only`伪类作用于只读的元素

`:read-write`伪类作用于可以编辑的元素

```html
<form>
  <input type="text" readonly value="lalala" />
</form>
<div contenteditable>
  <h1>Click on this text to edit it</h1>
</div>
```

```css
input:read-only{color: pink}
div:read-write{color: pink}
```



##### :required

`:required`伪类作用于含有`required`属性的元素

```html
<input type="email" required />
```

```css
input:required{ border: 1px solid orange; }
```



##### :scoped(实验性功能)

css作用域伪类[look at here](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:scope)



##### :dir(实验性功能)

`:dir`伪类目前只有firefox支持，我们需要使用dir HTML属性在标记中指定相关元素的方向性，目前有两个方向可用和支持：ltr（从左到右）和rtl（从右到左）。

```html
<div dir="ltr">ltr lllll<div>
```

```css
:-moz-dir(ltr) {color: pink}
```



##### :lang

`:lang`伪类用于html元素中有lang属性的

```html
<div lang="en">
  	gbbacy
</div>
```

```css
div:lang(en){ color: pink }
```



##### :root

`:root`伪类作用于html跟元素

```css
:root{
  background: gray
}
```



##### :fullscreen(实验性功能)

`:funllscreen`伪类在浏览器出发全屏时候起作用

```html
<h1 id="element">This heading will have a solid background color in full-screen mode.</h1>
<button onclick="var el = document.getElementById('element'); el.webkitRequestFullscreen();">Trigger full screen!</button>
```

```css
h1:-webkit-full-screen {
    background: orange;
}
```



##### ::before/:before ::after/:after

在介绍这些伪类之前需要了解什么是伪元素，伪元素是不存在dom当中，但是可以通过伪劣来操作就像操作正常的html元素一样。

```html
<h1>Ricardo</h1>
```

```css
h1:before {
    content: "Hello "; /* Note the space after the word Hello. */
}
```



##### ::backdrop(实验性功能)

全屏模式下起作用的伪类，[look at here](https://developer.mozilla.org/en-US/docs/Web/CSS/::backdrop)



#####  ::first-letter/:fist-letter ::first-line/:first-line

`:first-letter`伪类作用于第一个单词的第一个字母

`:first-line`伪类作用于第一行文字

```css
h1::first-letter { color: pink }
```

 

##### ::selection

`::selection`伪类作用于被选择中的元素

```css
::selection { background: pink }
```



#####  ::placeholder (实验性功能)

`::placeholder`属性作用于form表单中有placeholder属性的

```css
input::-moz-placeholder {
    color:#666;
}

input::-webkit-input-placeholder {
    color:#666;
}

/* IE 10 only */
input:-ms-input-placeholder {
    color:#666;
}

/* Firefox 18 and below */
input:-moz-input-placeholder {
    color:#666;
}
```



### 参考链接

CSS全部选择器：[Selectors from CSS4 TO CSS1](https://css4-selectors.com/selectors/)

CSS选择器打怪升级：[CSS Diner](http://flukeout.github.io/)

