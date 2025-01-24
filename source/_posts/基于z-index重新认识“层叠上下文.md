> z-index 是在设置层级的时候经常会用到的一个属性，很常用，好像也觉得没什么特别，在看过一些大佬的文章之后才知道原来“世界怎么大，我的知识这么点”

### 疑问

1. z-index 除了 position 能使它生效之外，还有什么属性可以让他生效？
2. 形成层叠上下文的节点，z-index 就能生效（误解）。
3. 子元素的层级就一定在父元素上面（父元素 Position 非 static）？

下面，带着疑问去整理相关的知识点。

### 基本用法

当需要设计一个层级的“层级”的时候

```
.box1{
    position:absolute；   //非static【默认】;absolute,fixed,relative
    z-index:2
}

//可能我box2 想在box1的上面
.box2{
    postion:absolute;
    z-index:3
}

//从之前的应用上来开，在让z-index生效的情况下，z-index 的值是越大表示层级越高【兄弟元素】
```

可能之前在对于“层级”的一个了解仅限于此，并没有去做更多详细的了解，但是其实这只是 css“层级”关系里面的冰山一角，更具体的，可能需要好好了解一下“层叠上下文”这个点。

### 层叠上下文是什么鬼？

层叠上下文是 HTML 元素的三维概念，这些 HTML 元素在一条假想的相对于面向（电脑屏幕的）视窗或者网页的用户的 z 轴上延伸，HTML 元素依据其自身属性按照优先级顺序占用层叠上下文的空间。

形成层级上下文的条件有很多，例如：

- 根元素（HTML）
- z-index 不为 auto,【position 非 static || display:flex/inline-flex ||display:grid】
- 固定（fixed） / 沾滞（sticky）定位（沾滞定位适配所有移动设备上的浏览器，但老的桌面浏览器不支持）
- opacity 属性值在 (0,1) 这个区间
- transform 不为 none
- filter 不为 none
- mix-blend-mode 不为 normal
- perspective 不是 none
- clip-path 不为 none
- mask/mask-image/mask-border 不为 none
- isolation 为 isolate
- 在 will-change 中指定任意 css 【能通过设置 will-change 对 css 动画进行优化】
- -webkit-overflow-scrolling 为 "touch"
- contain 为 layout 或者 pait

从上面列出来的属性中，我试着去推断一些信息

1. z-index 的方式是非常直接的,通过 z-index 的层级数字来去确定层级关系（在 zIndex 生效的情况下）
2. opacity,filter 这两个主要跟色彩的合成有关系，在不同层叠的情况下这两个属性会出现不同的结果（影响到了渲染）
3. trnasform,本来以为只有在涉及到 3d 转换的时候才会形成层叠上下文，测试的结果是`translate(0,0);`也会，那再重新查一下 trnasform 的一个描述“**_CSStransform 属性允许你旋转，缩放，倾斜或平移给定元素。这是通过修改==CSS 视觉格式化模型的坐标空间==来实现的_**。”

### 谁是老大

在有多个层叠上下文盒子的情况下，遵循一下规则

1. 在 z-index 生效的情况下，兄弟元素根据 z-index 的大小来决定谁在上面；
2. 两个盒子都形成层叠上下文，且==z-index:auto==,的情况下，根据 dom 节点的顺序来决定谁在上面（后面节点大于前面节点）。
3. 背景 < 布局 < 内容

### 测试

1.[zIndexDemo1](https://codepen.io/yagao/pen/JjPVmjK)，测试层叠上下文跟设置 position 盒子的层叠关系

```
//html
<div class="parent">
  <div class="Box1"></div>
  <div class="Box2">
    hello world
  </div>
<div>

//css
.Box1{
  position:absolute;
  left:0;
  top:0;
  background-color:pink;
  width:100%;
  height:100px;
}
.Box2{
  width:100%;
  height:40px;
  background-color:red;
}

//此处的结果是Box2被Box1覆盖掉了。Box1脱离了文档流浮在上方

调整 css

.Box2{
  width:100%;
  height:40px;
  background-color:red;
  opacity:0.99;
}

//Box2形成层叠上下文，Box2 覆盖了 Box1；

调整html
<div class="parent">
  <div class="Box2">
    hello world
  </div>
  <div class="Box1"></div>
<div>

//Box1覆盖着Box2；

```

结论：在 Box1 设置了定位之后，会覆盖住 Box2，但在 Box2 添加了 opacity:0.5 形成层叠上下文之后，Box2 跟 Box1 就在同一起跑线上了，这个时候就看 dom 的先后顺序来觉得谁覆盖谁。

2.[zIndexDemo2](https://codepen.io/yagao/pen/OJLGGbp)，测试 flex 子元素是否让 z-inde 生效

```
//html
<div class="parent">
  <div class="Box1"></div>
  <div class="Box2">
    hello world
  </div>
<div>

//css
.parent{
  display:flex;
}
.Box1{
  width:100px;
  height:100px;
  background-color:pink;
  z-index:2;
}
.Box2{h
  width:100px;
  height:100px;
  background-color:red;
  margin-left:-50px;
}
```

结论：flex 子元素是可以让 z-index 生效的。

3.[zIndexDemo3](https://codepen.io/yagao/pen/rNBgYBW) 子元素就一定在父元素（position 非 static）上面？

```
//html

<div class="parent">
  <div class="Box1"></div>
  <div class="Box2">
    hello world
  </div>
<div>

//css
.parent{
  background-color:yellow;
  padding:30px;
  position:relative;
  z-index:0;
}
.Box1{
  position:absolute;
  left:0;
  top:10px;
  background-color:pink;
  width:100%;
  height:100px;
  z-index:-1;
}
.Box2{
  width:100%;
  height:40px;
  background-color:red;
}

//Box1依然在parent上面

调整css
.parent{
  background-color:yellow;
  padding:30px;
  position:relative;
}

//去除parent的zIndex属性之后，Box1隐藏到了parent后面

调整css
.parent{
  background-color:yellow;
  padding:30px;
  position:relative;
  opacity:0.99;
}

//添加opacity:0.99 形成层叠上下文之后，Box1又无法通过zIndex负值隐藏到parent后面

调整css
.parent{
  background-color:yellow;
  padding:30px;
  position:fixed;
}

//结果与上面一致
```

结论：

- z-index:auto 不等于 z-index:0；
- 父元素在不形成层叠上下文的情况下，子元素是可以通过 zIndex 隐藏到父元素后面。

### 总结

1. 形成层叠上下文的方式有很多，更具体的看[MDN 的层叠上下文文档](https://developer.mozilla.org/zh-CN/docs/Web/Guide/CSS/Understanding_z_index/The_stacking_context)
2. 子元素可以通过 zIndex 负值隐藏在父元素后面，前提是父元素不形成层叠上下文
3. 可以让 zIndex 生效的不只是 postion，还有其他属性
4. 层级的覆盖
   - zIndex 的大小
   - 节点先后
   - 在没有脱离文档流的情况下。形成层叠上下文的节点会覆盖没有形成层叠上下文的节点。

**==备注：如果有错误的地方麻烦大佬多指点。==**
