# 样式书写顺序
![img.png](images/CSS样式书写顺序.png)

# 常见样式
1 伪类标签   :first-child  
2 元素模式转换  display: block、inline、inline-block  
3 文字垂直居中  line-height(等于盒子高度时是垂直居中)  
4 背景图片垂直居中  
```css
background-image: url(images/icon.png);
background-repeat: no-repeat;
background-position: left center;
```
5 超大图片做背景  
```css
background-image: url(images/bj.png);
background-repeat: no-repeat;
background-position: center top;
```
6 行内元素为了照顾兼容性，尽量只设置左右内外边距，不要设置上下内外边距，将其转换为块级元素和行内块元素就可以了  
7 CSS3新增样式  
```css
/*圆角边框*/
border-radius: 10px;
/*盒子阴影(阴影不占用空间，不会影响其他盒子排列)*/
box-shadow:
 /*文字阴影*/
text-shadow: 
```

# 盒子模型
### 水平居中  
①块级元素：给块级元素指定宽度和高度，然后
```css
margin: 0 auto;
```
②行内元素和行内块元素  
```css
/*给父元素添加如下属性*/
text-align:center;
```
### 嵌套块元素塌陷解决方案  
①可以为父元素定义上边框  
```css
border:1px solid transparent;
```
②可以为父元素定义上内边距  
③可以为父元素添加
```css
/*不会增加盒子大小*/
overflow:hidden;
```
### 清楚内外边距
布局前，清除网页元素的内外边距
```css
* {
    padding: 0;
    margin: 0;
}
```

# 网页布局
### 普通流
按照标签元素默认的排列方式进行排列

### 浮动流
①块级元素横向排列用浮动，浮动的盒子中间没有缝隙，是紧挨在一起的  
②浮动元素宽度由内容决定，不占有父元素的高度  
③具有行内块元素的特性

##### 浮动布局注意点
①先用标准流的父元素排列上下位置，之后内部子元素采取浮动排列左右位置  
②浮动的盒子只会影响浮动盒子后面的标准流，不会影响前面的标准流盒子

#### 清除浮动
①由于父级盒子很多情况下，不方便给定高度，但是子盒子浮动又不占有位置，最后导致父级盒子高度为0时，就会影响下面的标准流盒子  
②清除浮动的本质是清除浮动元素造成的影响，清除浮动之后，父级就会根据浮动的子盒子自动检测高度。父级有了高度，就不会影响下面的标准流了  

##### 清除浮动的方法
① 额外标签法：在浮动元素末尾添加一个空的标签，例如<div style="clear:both"></div>，或者其他块级元素标签  
② 父级添加overflow:hidden、auto、scroll  
③ :after伪元素
```css
.clearfix:after {
    content: "";
    display: block;
    height: 0;
    clear: both;
    visibility: hidden;
}

/* IE6、7专有 */
.clearfix{
    *zoom: 1;
}
```
④ 双伪元素清除浮动  
```css
.clearfix:before,.clearfix:after {
    content: "";
    display: table;
}

/* 照顾低版本浏览器 */
.clearfix:after {
    clear: both;
}
.clearfix {
    *zoom: 1;
}
```

### 定位
定位则是可以让盒子自由的在某个盒子内移动位置或者固定屏幕中某个位置，并且可以压住其他盒子  
定位 = 定位模式 + 边偏移

##### 静态定位
静态定位是默认形式，等同于标准流，没有边偏移

##### 相对定位
①移动位置的时候参照点是自己原来的位置  
②不脱标，继续保留原来位置

##### 绝对定位
①如果没有祖先元素或者祖先元素没有定位，则以浏览器为准定位（Document文档）  
②如果祖先元素有定位（相对、绝对、固定定位），则以最近一级的有定位祖先元素为参考点移动位置  
③绝对定位脱标不占有原来的位置  

常用场景：  
①子绝父相  
②绝对定位的盒子居中展示
```
加了绝对定位的盒子不能通过margin: 0 auto进行水平居中展示，但是可以通过一下计算方法实现水平和垂直居中  
① left:50%              让盒子的左侧移动到父级元素的水平中心位置  
② margin-left: -100px   让盒子向左移动自身宽度的一半
```

##### 固定定位
①以浏览器的可视窗口为参照点移动元素  
②跟父元素没有任何关系  
③不随滚动条滚动  
④固定定位不占有原先的位置  

##### 粘性定位
①以浏览器的可视窗口为参照点移动元素  
②粘性定位占有原先的位置  
③必须添加top、left、right、bottom其中一个才有效  
注意：跟页面滚动搭配使用，兼容性较差，而且IE不支持  

### 定位总结
①浮动元素不同，只会压住下面标准流的盒子，但是不会压住下面标准流盒子里面的文字，其产生的最初目的就是为了做文字环绕效果的  
②绝对定位（或者固定定位）会压住下面标准流所有的内容  
③相对定位可以直接通过margin:auto实现水平居中，而绝对定位和固定定位需要通过一定的计算间接实现居中效果  

# vertical-align属性应用
图片底部和div会有一个空白缝隙，原因是行内块元素会和文字的基线对齐，图片、表单等属于行内块元素，默认的vertical-align是基线对齐，此时可以给图片、表单等这些行内块元素的vertical-align
属性设置为middle，就可以让文字和图片垂直居中对齐
解决办法：  
```css
vertical-align: middle|top|bottom等
```

# 单行文本溢出显示省略号
```css
/* normal表示如果文字显示不开自动换行；nowrap表示强制一行内显示 */
white-space: normal|nowrap;
/* 溢出的部分隐藏 */
overflow: hidden;
/* 文字溢出的时候用省略号来显示 */
text-overflow: ellipse;
```

# 多行文本溢出显示省略号

# margin负值巧妙运用
让每个盒子margin往左侧移动-1px正好压住相邻盒子边框

# 伪元素选择器
伪元素选择器可以帮助我们利用CSS创建新标签元素，而不需要HTML标签，从而简化HTML结构  
①::before   在元素内部的前面插入内容  
②::after    在元素内部的后面插入内容  
```
before和after创建一个元素，但是属于行内元素  
新创建的这个元素在文档树中是找不到的，所以我们称为伪元素  
语法是：element:before{}  
before和after必须有content属性  
before在父元素内容的前面创建元素，after在父元素内容的后面插入元素  
伪元素选择器和标签选择器一样，权重为1
```













