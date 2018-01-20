### Flex 弹性盒子完整指南(翻译)

原文链接 [《A Complete Guide to Flexbox》](https://css-tricks.com/snippets/css/a-guide-to-flexbox/)

#### 一、背景

FlexBox布局模块的旨在提供一种更有效的方式去布局，在父容器中对齐和分配空间给子元素， 即使子元素的大小是未知或者是动态的。

Flex布局背后主要的思想就是让容器有能力改变子元素的宽度/高度以及顺序以最好的方式来填充可用空间(用来适应各种屏幕的尺寸)，flex容器可以扩展子元素去填充可用空间，或者缩小子元素防止溢出。

最重要的是，不同于常规的布局（基于水平方向的块级元素和基于水平的行内元素），flex布局属于方向不可知的，虽然常规布局也能实现页面布局，但是在遇到大型或者负责的项目时缺乏灵活性(特别是在改变方向，调整大小，拉伸，缩小等方面)

**提示**：Flexbox布局适合应用程序的组件小规模布局，而Grid布局适合大规模的布局



#### 二、基础和术语

由于FlexBox是一个完整的模块而不是单一的属性，所以他涉及到很多东西，包括整套属性。其中一些属性是为了在Flex容器上设置(父元素，称为'弹性容器')，一些属性是设置在子元素上面。

如果常规布局基于块级或者内联元素流动方向，而弹性盒子则基于"flex-流动方向"。请看一下下图中的规范，可以解释flex布局背后的思想。

![flex.png](http://oznz1caxs.bkt.clouddn.com/flexbox.png)

基本来说，子元素将根据主轴（从main-start - main-end）或者横轴（从cross-start - cross-end）来布置项目

- **main axis**(主轴)：主轴是弹性容器中子元素布局的主要轴。需要注意的是，它的方向并不一定是水平方向固定，这取决于flex-direction属性的取值
- **main-start | main-end**：子元素从main-start 到main-end在父容器中放置
- **main-size**(主尺寸)：A flex item's width or height, whichever is in the main dimension, is the item's main size. The flex item's main size property is either the ‘width’ or ‘height’ property, whichever is in the main dimension.
- **cross axis**(横轴)：垂直于主轴称为横轴，他的方向取决于主轴的方向
- **cross-start | cross-end**：子元素线性填充在flex中，从容器cross-start一端并朝着cross-end一端放置
- **cross size** - The width or height of a flex item, whichever is in the cross dimension, is the item's cross size. The cross size property is whichever of ‘width’ or ‘height’ that is in the cross dimension.



#### 作用于父容器属性

![flex-container](http://oznz1caxs.bkt.clouddn.com/flex-container.svg)

**display**：这定义了一个弹性容器，块级或者内联取决于给定的值，它为子元素提供了一个flex盒子容器

```css
.container{
  display: flex; /* or inline-flex */
}
/* 注意：定义为flex，css列将不会对flex容器起作用 */
```



**flex-direction**：这个属性建立了主轴，因此定义了子元素在父容器中的排列的方向，flex布局是单向布局（除了可选的wrap属性）flex容器中的元素可以想象成布置于水平或者垂直列中

![flex-dirction2](http://oznz1caxs.bkt.clouddn.com/flex-direction2.svg)

```css
.container{
  flex-direction: row | row-reverse | column | column-reverse
}
```

例子：[flex-direction](https://github.com/neverthanmore/flex-demos/tree/master/flex-direction)

- row(默认)：从左到右是ltr, 从右到左是rtl
- row-reverse：从右到左是ltr, 从左到右是rtl
- column：和rows属性一样，但是是从顶部到底部
- column-reverse：和row-reverse属性一样，但是是从底部到顶部



**flex-wrap**：默认情况下，flex盒子内元素会在一行上排列，但是可以通过这个属性允许他换行

![flex-wrap](http://oznz1caxs.bkt.clouddn.com/flex-wrap.svg)

```css
.container{
  flex-wrap: nowrap | wrap | wrap-reverse;
}
```

例子：[flex-wrap](https://github.com/neverthanmore/flex-demos/tree/master/flex-wrap)

- **nowrap**(默认)：所有flex元素在一行内填充
- **wrap**：溢出时flex元素会从上到下分多行填充
- **wrap-reverse**：溢出时flex元素会从下到上分多行填充



 **flex-flow**：fle-direction 和 flex-wrap的集合，默认为row nowrap

```css
flex-flow: <‘flex-direction’> || <‘flex-wrap’>
```



**justify-content**：这个属性定义了在主轴上的对齐方式，当弹性盒里一行上的所有子元素都不能伸缩或已经达到其最大值时，它可以对多余的空间进行分配。当元素溢出时，仍然能够在对齐上进行控制

[justify-content展示图片较大,通过外链访问](http://oznz1caxs.bkt.clouddn.com/justify-content-2.svg)

```css
.container {
  justify-content: flex-start | flex-end | center | space-between | space-around;
}
```

例子：[justify-content](https://github.com/neverthanmore/flex-demos/tree/master/justify-content)

- **flex-start**(默认)：各行向弹性盒容器的起始位置堆叠。弹性盒容器中第一行的侧轴起始边界紧靠住该弹性盒容器的侧轴起始边界，之后的每一行都紧靠住前面一行

- **flex-end**：各行向弹性盒容器的结束位置堆叠。弹性盒容器中最后一行的侧轴起结束界紧靠住该弹性盒容器的侧轴结束边界，之后的每一行都紧靠住前面一行

- **center**：各行向弹性盒容器的中间位置堆叠。各行两两紧靠住同时在弹性盒容器中居中对齐，保持弹性盒容器的侧轴起始内容边界和第一行之间的距离与该容器的侧轴结束内容边界与第最后一行之间的距离相等。（如果剩下的空间是负数，则各行会向两个方向溢出的相等距离。）

- **space-between**：各行在弹性盒容器中平均分布

- **space-around**：各行在弹性盒容器中平均分布，，两端保留子元素与子元素之间间距大小的一半

  ​

**align-items**：这个属性定义了弹性元素在横轴上的排列方式，可以把这个属性看成是justify-content在横轴上排列的方式

![align-items](http://oznz1caxs.bkt.clouddn.com/align-items.svg)

```css
.container {
  align-items: flex-start | flex-end | center | baseline | stretch;
}
```

例子：[align-items](https://github.com/neverthanmore/flex-demos/tree/master/align-items)

- **flex-start**：弹性元素在横轴方向从顶部开始排列

- **flex-end**：弹性元素在横轴方向从底部开始排列

- **center**：弹性元素在横轴方向从中间开始排列

- **baseline**：弹性元素以基线对齐

- **stretch**(默认)：弹性元素拉伸填满容器

  ​

**align-content**：当横轴上超过一行，这个属性会将弹性元素排列，就像justify-content在主轴上对当行元素进行排列一样

![align-content](http://oznz1caxs.bkt.clouddn.com/align-content.svg)

```css
.container {
  align-content: flex-start | flex-end | center | space-between | space-around | stretch;
}
```

例子：[align-content](https://github.com/neverthanmore/flex-demos/tree/master/align-content)

- **flex-start**：各行元素向横轴顶部开始排列
- **flex-end**：各行元素向横轴顶部开始排列
- **center**：各行元素在弹性盒容器的中间位置堆叠
- **space-between**：各行元素在弹性盒容器中平均分布
- **space-around**：各行元素在弹性盒容器中平均分布，两端保留子元素与子元素之间间距大小的一半
- **stretch**(默认)：弹性元素延伸占据剩余空间



#### 作用于子元素属性

![flex-items](http://oznz1caxs.bkt.clouddn.com/flex-items.svg)

**order**：默认情况下，弹性元素会按照代码顺序排列，order属性可以控制弹性元素在容器中排列的顺序。

![order](http://oznz1caxs.bkt.clouddn.com/order-2.svg)

```css
.item {
  order: <integer>; /* default is 0 */
}
```

例子：[order](https://github.com/neverthanmore/flex-demos/tree/master/order)



**flex-grow**：这属性定义了弹性元素在容器中可扩展的能力，它接收一个无单位的值用来提供一个比例。他规定了弹性元素可以在弹性容器内的可用空间量。假如弹性元素的flex-grow的属性值都是1，弹性容器的剩余空间都会被子元素均匀分配，如果其中的一个子元素的flex-grow值为2，剩余的空间中，他占据的空间将会是其他子元素的两倍。

![flex-grow](http://oznz1caxs.bkt.clouddn.com/flex-grow.svg)

```css
.item {
  flex-grow: <number>; /* default 0 */
}
/* 负数是无效的 */
```

例子：[flex-grow](https://github.com/neverthanmore/flex-demos/tree/master/flex-grow)



**flex-shrink**：属性定义了弹性元素在容器中收缩的能力

```css
.item {
  flex-shrink: <number>; /* default 1 */
}
```

例子：[flex-shrink](https://github.com/neverthanmore/flex-demos/tree/master/flex-shrink)



**flex-basis**：属性定义了在剩余控件分配前元素默认大小，他可以是一个长度(例如20%或者5rem等)或者关键字。auto关键字意思是"看我的高度或者宽度属性"（这是临时由main-size关键字完成，直到弃用）。

```css
.item {
  flex-basis: <length> | auto; /* default auto */
}
```



**flex**：这是flex-grow，flex-shrink和flex-basis结合的简写。 第二个和第三个参数（flex-shrink和flex-basis）是可选的。 默认值是0 1 auto。

```css
.item {
  flex: none | [ <'flex-grow'> <'flex-shrink'>? || <'flex-basis'> ]
}
```



**align-self**：这允许为各个弹性项目覆盖默认对齐方式（或由align-items指定的对齐方式）。

![align-self](http://oznz1caxs.bkt.clouddn.com/align-self.svg)

```css
.item {
  align-self: auto | flex-start | flex-end | center | baseline | stretch;
}
```

例子: [align-self](https://github.com/neverthanmore/flex-demos/tree/master/align-self)

每个属性的解释参照align-items



