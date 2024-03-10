### 前言

前些天被人问到了 BFC ，在回答的时候俺支支吾吾半天只能说个大概，没办法有条理性的告诉他 BFC 是什么有什么用。查阅资料，发现也没有能够从源头上系统化梳理BFC的文章，遂自己查阅文档写出本文。

### 什么是BFC？

BFC 全称是`Block Formatting Context`，即**块格式化上下文**。它是 CSS2.1 规范定义的，关于 CSS 渲染定位的一个概念。要明白 BFC 到底是什么，还得首先聊聊 CSS 中的**视觉格式化模型**。

### 什么是视觉格式化模型、什么是盒子类型

写了这么久Web的我们都知道：块级元素换行、行内元素不换行、浮动和绝对定位脱离文档流等等。
以上都代表了浏览器对某种节点类型的处理方式，这些处理方式就是由**视觉格式化模型**定义的。

**视觉格式化模型**，即 `Visual Formatting Model`，是用来处理文档并将它显示在视觉媒体（浏览器、移动端）上的机制，它也是CSS中的一个概念，或者说，**一种规则**。

那么它是如何工作的呢?

> In the visual formatting model, each element in the document tree generates zero or more boxes according to the box model.

首先，**视觉格式化模型**定义了**盒子**（Box）这个概念，————文档树中的每个元素，都属于某种盒子。它可用于文档元素的**定位、布局和格式化**，盒子类型主要包括了：

1. 块盒子
2. 行内盒子
3. 匿名盒子（没有名字不能被选择器选中的盒）

这边的术语非常多，而且不能乱用，[具体可参考MDN文档](https://developer.mozilla.org/zh-CN/docs/Web/CSS/Visual_formatting_model) 。以下只提取最大级的概念。

盒子的类型由元素的 `display`属性决定：

```
display: block;//list-item  table
display: inline;//inline-block  inline-table
display: inherit;
```

#### 块盒子 Block Box

1. 当元素的 `display`为 `block`、`list-item` 或 `table` 时，该元素将成为**块级元素**( block-level element )。
2. 每个**块级元素**都会至少生成一个**块级盒子**（ block-level box ）。
3. 每个**块级盒子**都会参与[块格式化上下文](https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_display/Block_formatting_context)的创建，即BFC。
4. 还有一种类型：**块容器盒子**（ block container box ）：它要么只包含其他**块级盒子**，要么只包含**行内盒子**并同时创建一个[行内格式化上下文](https://developer.mozilla.org/zh-CN/docs/CSS/Inline_formatting_context)，即IFC。。
5. 块盒子 = 既是**块级盒子**又是**块容器盒子**的盒子。

简单理解：每个**块级元素**至少生成一个**块级盒子**，参与**BFC**，在视觉上表现为：呈现为块，竖直排列。

#### 行内盒子 Inline Box

1. 如果一个元素的 `display`属性为 `inline`、`inline-block` 或 `inline-table`，则称该元素为**行内级元素**( inline-level element )。
2. **行内级元素**生成**行内级盒子** （ inline-level box ）。
3. **行内级盒子**可能会参与[行内格式化上下文（ inline formatting context ）](https://developer.mozilla.org/zh-CN/docs/CSS/Inline_formatting_context)的创建，即IFC。
4. **行内盒子** = 参与了行内格式化上下文创建的**行内级盒子**。 
5. **原子行内级盒**(atomic inline-level boxes) = 没参与IFC创建的行内级盒子，如`display`为 `inline-block` 或 `inline-table`。

简单理解：**行内级元素**会生成**行内盒子**，参与**IFC**，在视觉上表现为：将内容与行内级元素排列为多行；例如包含文本、图片等多种行内级元素的段落。

#### 匿名盒子 Anonymous Box

1. 在某些情况下进行视觉格式化时，需要添加一些增补性的盒子，这些盒子**不能用 CSS 选择符**选中，因此称为**匿名盒子** 。
2. 此时它的所有可继承的 CSS 属性值都为 `inherit`。
3. 它也分为匿名块盒与匿名行内盒

简单理解：某些元素（如纯文本元素），在视觉呈现的时候，会自动创建出一个不能用CSS选择符选中的盒子，就称这个盒子为匿名盒子。

![boxes_types.png][1]

<font id="postionProject"></font>
### 浏览器根据盒子类型形成定位方案

浏览器在进行视觉呈现的时候，主要步骤分为：生成盒子 -> 形成**定位方案** -> 完成布局。

**定位方案**指的就是定位盒子的方案，它又分为三种：

1. 普通流。
2. 浮动。
3. 绝对定位。

#### 普通流 Normal flow

1. 当 CSS 的 `position` 属性为 `static` 或 `relative`，并且 `float` 为 `none` 时，其布局方式为普通流。
2. 在普通流中，会按照次序依次定位每个盒子：
   1. 在 BFC 中，盒子在**垂直方向**依次排列。
   2. 在 IFC 中，盒子在**水平方向**依次排列
3. 当`position` 为 `static` 时，为静态定位，盒子位置 = 普通流布局中位置。
4. 当 `position` 为 `relative` 时，为相对定位，盒子位置 = 普通流布局中位置 +  `top`、`bottom`、`left`和 `right` 各个属性产生的偏移量。但仍然**占据原有的空间 ，其它常规流不能占用这个位置**。

#### 浮动 Floats

1. 一个盒子的 `float` 值不为 `none`，并且其 `position` 为 `static` 或 `relative` 时，该盒子为浮动定位。
2. 在浮动定位中，盒子会浮动到当前行的开始或尾部位置。
3. 其他普通流定位的盒子会**环绕在它的周边，除非清除浮动**


#### 绝对定位 Absolute positioning

1. 如果元素的 `position` 为 `absolute` 或 `fixed`，该元素为绝对定位。
2. 对于 `position: absolute`，元素定位将相对于最近的一个 `relative`、`fixed`或 `absolute`的父元素，如果没有则相对于 `body`。
3. 对于 `position: fixed`，元素定位将相对于**屏幕视口**（viewport）的位置，且不会因为屏幕滚动而改变。
4. 在绝对定位中，盒子会**完全从当前常规流中移除**，并**不占据原有的空间，**
5. 盒子位置 = 定位起点 + `top`，`bottom`，`left` 和 `right` 各个属性产生的偏移



### 回到块格式化上下文

经过上面概念的梳理，我们现在知道什么是 BFC 了————它是 Web 页面 CSS 视觉渲染的一部分， 用于**决定块盒子的布局** 以及 **浮动相互影响范围** 的一个区域 。

有一句话我觉得概括得很好———— ** BFC 就是 CSS 里的块级作用域。总之它是一个范围、一个框、一个区域。**



#### 哪些元素会生成BFC：

1. **根元素**。即`HTML`元素。
2. **浮动元素**。`float`的值不为`none`。
3. **绝对定位元素**。`position`的值为`absolute`或`fixed`。
4. `overflow`的值不为`visible`。
5. **行内块元素**或者**表格单元格**。 `display:inline-block`、`table-cell`或 `table-caption`。
6. **弹性盒元素**。`display: flex`或 `inline-flex`

最常见的是 `overflow:hidden`、`display: flex`、`float:left/right`、`position:absolute`。也就是说，每次看到这些属性的时候，就代表了该元素已经创建了一个 BFC 了。




#### BFC 的范围

BFC 的范围在 MDN 中是这样描述的。

> A block formatting context contains everything inside of the element creating it that is not also inside a descendant element that creates a new block formatting context.

意思是一个 BFC 包含创建该上下文元素的所有子元素，但不包括创建了新 BFC 的子元素的内部元素。

这段看上去有点奇怪，我是这么理解的，加入有下面代码，`class`名为 `.BFC`代表创建了新的 BFC ：
 
```
<div id='div_1' class='BFC'>
    <div id='div_2'>
        <div id='div_3'></div>
        <div id='div_4'></div>
    </div>
    <div id='div_5' class='BFC'>
        <div id='div_6'></div>
        <div id='div_7'></div>
    </div>
</div>
```



这段代码表示，`#div_1`创建了一个块格式上下文，这个上下文包括了 `#div_2`、`#div_3`、`#div_4`、`#div_5`。即 `#div_2`中的子元素也属于 `#div_1`所创建的BFC。但由于 `#div_5`创建了新的BFC，所以 `#div_6`和 `#div_7`就被排除在外层的BFC之外。

我认为，这从另一方角度说明， **一个元素不能同时存在于两个BFC中** 。

BFC 的一个最重要的效果是，让处于 BFC 内部的元素与外部的元素**相互隔离**，使内外元素的**定位不会相互影响**。

所以，如果一个元素能够同时处于两个 BFC 中，那么就意味着这个元素能与两个 BFC 中的元素发生作用，就违反了 BFC 的隔离作用，所以这个假设就不成立了。

#### BFC 的布局规则：

1. 内部的盒子会在**垂直方向**上一个接一个的放置。（相当于内部也有一个常规流）
2. 盒子垂直方向的距离由`margin`决定。属于同一个 BFC 的两个**相邻盒子**的`margin`会发生重叠（即`margin塌陷`）。
3. 每个元素的`margin左边界`，都与**容器块**的**左边界**相接触。即使是浮动元素也是如此。
4. BFC 就是页面上的**一个隔离的独立容器**，容器里面的子元素不会影响到外面的元素，反之亦然。
5. 计算 BFC 的高度时，考虑 BFC 所包含的所有元素，连浮动元素也参与计算。
6. 浮动盒区域不叠加到 BFC 上。


看到以上的几条约束，想想我们学习 CSS 时的几条规则

1. 块级元素会扩展到与父元素同宽，所以块级元素会垂直排列。
2. 垂直方向上的两个相邻块级元素的`margin`会塌陷，而水平方向不会。
3. 浮动元素会尽量接近往左上方（或右上方）。
4. 为父元素设置 `overflow：hidden`或 `float:left`，则会包含浮动元素。

#### BFC的作用

1. 防止`margin`塌陷，包括垂直相邻塌陷、嵌套元素塌陷。
2. 清除内部浮动，防止高度塌陷，或者防止浮动元素与其他元素重叠。
3. 自适应双栏、多栏布局

### Demo1：防止margin塌陷

#### 问题描述：
M3C文档里是这样描述`margin塌陷`的:

> In CSS, the adjoining margins of two or more boxes (which might or might not be siblings) can combine to form a single margin. Margins that combine this way are said to collapse, and the resulting combined margin is called a collapsed margin.


在一个 BFC 里，两个或多个**相邻盒子**的上外边距和下外边距可能会塌陷（折叠）为一个外边距，其大小会取其中外边距值大的那个，这种现象就是`margin塌陷`。需要注意的是，**浮动的元素**和**绝对定位**这种脱离文档流的元素的外边距不会折叠，<a href="#postionProject">因为他们属于其他两种的定位方案</a>。**`margin塌陷`只会出现在垂直方向**。

<font id="adjacent"></font>
W3C文档对于**“相邻”**（ adjoining ）的定义如下：
> Two margins are adjoining if and only if:
> - both belong to in-flow block-level boxes that participate in ** the same block formatting context**
> - **no line boxes, no clearance, no padding and no border separate them** (Note that certain zero-height line boxes (see 9.4.2) are ignored for this purpose.)
> - both belong to vertically-adjacent box edges, i.e. form one of the following pairs:
>     - top margin of a box and top margin of its first in-flow child
>     - bottom margin of box and top margin of its next in-flow following sibling
>     - bottom margin of a last in-flow child and bottom margin of its parent if the parent has 'auto' computed height
>     - top and bottom margins of a box that does not establish a new block formatting context and that has zero computed 'min-height', zero or 'auto' computed 'height', and no in-flow children
>
>A collapsed margin is considered adjoining to another margin if any of its component margins is adjoining to that margin.

简单概括：两者属于同一个 BFC 且它两之间没有行盒子，没有间隙，没有`padding`也没有`border`，那他们之间就属于**相邻**。

#### 计算原则：
出现`margin塌陷`时合并外边距的计算原则如下：
● 如果两者都是正数，那么就取**最大值**。
● 如果是一正一负，就会取正值减去负值的绝对值。
● 两个都是负值时，用 0 减去两个数中绝对值大的那个。

#### 问题1：兄弟之间发生margin塌陷
```
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
    <style>
      * {
        margin: 0;
        padding: 0;
      }
      .first {
        margin: 100px;
        background: lightgreen;
        border: 1px solid;
        width: 100px;
        height: 100px;
      }
      ul {
        /* display: inline-block; */
        /* position: fixed; */
        /* float: left; */
        margin: 100px;
        background: lightblue;
        border: 1px solid;
        width: 200px;
      }
      li {
        margin: 10px 20px;
      }
    </style>
  </head>

  <body>
    <div class="first"></div>
    <ul>
      <li>item1</li>
      <li>item2</li>
      <li>item3</li>
    </ul>
  </body>
</html>

```
如图，以上代码创建两个相邻的块级元素，并都将他们的`margin`设置为`100px`。可以观察到两个块级元素发生`marin塌陷`。
![demo1_1.jpg][2]

##### 解决办法：
1.利用绝对定位、浮动盒子、`inlne-block`盒子都不会发生`margin塌陷`的特性。[参考CSS2文档][3]
> - Margins between a **floated** box and any other box do not collapse (not even between a float and its in-flow children).
> - Margins of elements that establish new block formatting contexts (such as floats and elements with 'overflow' other than 'visible') do not collapse with their in-flow children.
> - Margins of **absolutely** positioned boxes do not collapse (not even with their in-flow children).
> - Margins of **inline-block** boxes do not collapse (not even with their in-flow children).

设置下方元素`float:left`和`position:absolute`使下方元素脱离普通流，或设置'display:inline-block'使其成为`inline-block`盒子。

**注意**：如果此时只是给`<ul>`设置`overflow:hidden`，`display:flex`，` display:table-cell `等，使其生成 BFC 都不会发生作用。因为此时虽然**对内已经生成了BFC**，但是对外它还是为一个**块级盒子**，两相邻**块级盒子**之间就是会`margin塌陷`。<a href="#adjacent">回去看看“相邻”的定义</a>

> Vertical margins between adjacent block-level boxes in a block formatting context collapse.
![demo1_2.jpg][4]

2.在底部元素外部包裹一个新的元素，使其生成 BFC 。利用`margin塌陷`只会发生在同一个 BFC 下的两相邻**块级盒子**之间的特性。
![demo1_3.jpg][5]

#### 问题2：父子之间发生margin塌陷
其实上面说的，相邻兄弟**块级盒子**之间会发生`margin塌陷`是有利于我们实际开发的，一般可以不修改，而这个发生在父子之间的，几乎就是必改的了。
```
...
ul {
        /* display: inline-block; */
        /* position: fixed; */
        /* float: left; */
        margin: 100px;
        background: lightblue;
        /* 注释下一行，会发生父子间margin塌陷 */
        /* border: 1px solid; */
        width: 200px;
    }
```
如代码，我们注释掉`<ul>`元素的`boder`，此时会发生父子块级盒子之间的`margin塌陷`。如图，我们给每个`<li>`设置了`margin`，但是会发现此时`<div>`与`<ul>`之间的垂直距离，取`<div>、<ul>、<li>`三者之间的最大外边距,为`100px`。

![demo1_4.jpg][6]

为什么注释掉`<ul>`元素的`boder`就会引起这个现象呢？？因为注释掉的话，`<div>`和`<li>`之间就没有任何阻隔了，根据上面提到的**相邻**两个或多个**盒子**之间就会产生`margin塌陷`的规则，就会出现这个现象。<a href="#adjacent">回去看看“相邻”的定义</a>

##### 解决方法:
1. 给`<ul>`添加`boder`、`padding`等属性，使`<div>、<ul>、<li>`三者不满足“相邻”条件。
![demo1_5.jpg][7]

2.使`<ul>`内部变成 BFC，利用 BFC 之间互不影响的特性。

![demo1_6.jpg][8]


### Demo2：防止高度塌陷或防止浮动元素与其他元素重叠。

#### 问题1：高度塌陷
```
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title></title>
      <style>
    * {
        margin: 0;
        padding: 0;
      }

    .parent {
        border: 2px solid black;
        width: 400px;
        /* overflow: hidden; */
    }
 
    .child1 {
        float: left;
        background-color: lightblue;
        width: 100px;
        height: 300px;
    }
    .child2 {
        background-color: lightgreen;
        width: 300px;
        height: 100px;
    }
</style>
<body>
    <div class="parent">
        <div class="child1"></div>
        <div class="child2"></div>
    </div>
</body>

</html>

```
如图，以上代码在一个`父<div>`里创建了两个`子<div>`，并将第一个`子<div>`设置为浮动。可以观察到`父<div>`的高度并没有计算浮动`子<div>`的高度，发生了`高度塌陷`。

![demo2_1.jpg][9]

##### 解决办法：
使`父<div>`生成 BFC 即可，利用：**BFC 计算高度时，考虑 BFC 所包含的所有元素（不只是子元素），连浮动元素也参与计算**这一特性。就如上一张图中的`HTML`元素，因为它也是BFC，所以能计算到浮动元素的高度。

当然清除浮动也可以使用一个包含`clear:both`的空元素，或者设置`父<div>`的伪元素`:after`为`display:block;clear:both`来实现，相比BFC，设置伪元素效果更好，更多人使用。

![demo2_2.jpg][10]

#### 问题2：浮动元素与其他元素重叠
此时，观察图片，我们还发现两个`子<div>`之间还会发生重叠，这是因为浮动元素会**脱离普通流**并不占据原本空间，而其他普通流定位的盒子会**环绕在它的周边**。

![demo2_3.jpg][11]

##### 解决办法：
使发生重叠的`子<div>`生成 BFC 即可，利用：**浮动盒区域不叠加到 BFC 上**这一特性。

![demo2_4.jpg][12]


### Demo：自适应双栏、三栏布局。
```
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>自适应双栏布局和三栏布局</title>
    <style>
        * {
            margin: 0;
            padding: 0;
        }

        .aside {
            float: left;
            width: 100px;
            min-height: 300px;
            background: lightcoral;
        }

        .main {
            height: 200px;
            background: lightgreen;
            display: flex;
            overflow: hidden;
        }

        .left {
            background: pink;
            float: left;
            width: 180px;
        }

        .center {
            background: lightyellow;
            overflow: hidden;

        }

        .right {
            background: lightblue;
            width: 180px;
            float: right;
        }

        .BFC {
            overflow: hidden;
        }

        .item {
            width: 50px;
            height: 50px;
            margin: 20px;
            background-color: lightgray;

        }
    </style>


</head>

<body>

    <section class="BFC">
        <div class="aside">我是aside</div>
        <div class="main">
            <div class="item"></div>
            <div class="item"></div>
            <div class="item"></div>
        </div>
    </section>

    <section class="BFC" style="margin-top: 100px;">
        <div class="container">
            <div class="left">
                <pre>
      .left{
        background:pink;
        float: left;
        width:180px;
      }
          </pre>
            </div>
            <div class="right">
                <pre>
      .right{
        background:lightblue;
        width:180px;
        float:right;
      }
          </pre>
            </div>
            <div class="center">
                <pre>
      .center{
        background:lightyellow;
        overflow:hidden;
        height:116px;
      }
          </pre>
            </div>

    </section>
</body>

</html>
```
原理上文已经说过了，这里就不再重复了，我也写在了图中。
![demo3_1.jpg][13]
![demo3_2.jpg][14]


### 总结
至此，我们对BFC的学习就结束啦。在写这篇文章的过程中，我才发现原来外边距塌陷和BFC并不可以混淆而谈，外边距塌陷有它自己非常明显的充分必要条件，它的发生与**“相邻”的盒子**有关，而 BFC 只是解决它的一个办法而已，并不是只要生成 BFC 就能解决。这篇文章还是查阅了非常多资料的，上面的Demo也都可以在我的git地址中找到，如果恰好对你们有所帮助的话请不要吝啬你们的点赞和收藏哦~谢谢。



  [1]: http://120.25.166.245/usr/uploads/2024/03/3873987992.png
  [2]: http://120.25.166.245/usr/uploads/2024/03/3922133295.jpg
  [3]: https://www.w3.org/TR/CSS21/box.html#collapsing-margins
  [4]: http://120.25.166.245/usr/uploads/2024/03/371289040.jpg
  [5]: http://120.25.166.245/usr/uploads/2024/03/3527031881.jpg
  [6]: http://120.25.166.245/usr/uploads/2024/03/2482920802.jpg
  [7]: http://120.25.166.245/usr/uploads/2024/03/774208045.jpg
  [8]: http://120.25.166.245/usr/uploads/2024/03/3799098106.jpg
  [9]: http://120.25.166.245/usr/uploads/2024/03/1072714288.jpg
  [10]: http://120.25.166.245/usr/uploads/2024/03/1469980201.jpg
  [11]: http://120.25.166.245/usr/uploads/2024/03/4047192120.jpg
  [12]: http://120.25.166.245/usr/uploads/2024/03/2199067581.jpg
  [13]: http://120.25.166.245/usr/uploads/2024/03/2900843260.jpg
  [14]: http://120.25.166.245/usr/uploads/2024/03/750709272.jpg