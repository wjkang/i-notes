
```
flex-wrap: nowrap | wrap | wrap-reverse;
nowrap（默认）：不换行。
wrap：换行，第一行在上方。
wrap-reverse：换行，第一行在下方。

项目在主轴上的对齐方式。
justify-content: flex-start | flex-end | center | space-between | space-around;
flex-start（默认值）：左对齐
flex-end：右对齐
center： 居中
space-between：两端对齐，项目之间的间隔都相等。
space-around：每个项目两侧的间隔相等。所以，项目之间的间隔比项目与边框的间隔大一倍。

定义项目在交叉轴上如何对齐。
align-items: flex-start | flex-end | center | baseline | stretch;
flex-start：交叉轴的起点对齐。
flex-end：交叉轴的终点对齐。
center：交叉轴的中点对齐。
baseline: 项目的第一行文字的基线对齐。
stretch（默认值）：如果项目未设置高度或设为auto，将占满整个容器的高度。
```

[flex](http://static.vgee.cn/static/index.html ':include :type=iframe width=100% height=16200')

<vuep template="#example"></vuep>

<script v-pre type="text/x-template" id="example">
  <style>
  .container{
  display:flex;
  width:500px;
  background-color:silver;
}
.container div{
  height:100px;
  line-height:100px;
  text-align:center;
}
.item1{
  background-color:#94a04a;
  flex-basis:auto;/*默认为auto,auto与实际内容有关,设置为非0,按flex-grow填充剩余空间的时候，占据的空间要多加上flex-basis的数值，设为auto，占据的空间要多加上“多出的内容”。如果内容多出占用的空间，则会撑开当前项目，其它项目被压缩*/
  flex-grow:1;
}
.item2{
  background-color:#56b8ef;
  flex-grow:1;
  flex-basis:100px;
  flex-shrink:8;/*flex容器宽度不够时，被压缩的系数，数值越大被压缩越多*/
}
.item3{
  background-color:#48b38a;
  flex-grow:1;
  flex-basis:100px;
  flex-shrink:1;
}
</style>
<template>
	<div class="container">
    <div class="item1">1多出的内容多出的内容多出的内容多出的内容多出</div>
    <div class="item2">2</div>
    <div class="item3">3</div>
  </div>
</template>

  <script>
    module.exports = {
      data: function () {
        return { name: 'Vue' }
      }
    }
</script>
</script>

[https://www.w3cplus.com/css3/a-visual-guide-to-css3-flexbox-properties.html](https://www.w3cplus.com/css3/a-visual-guide-to-css3-flexbox-properties.html)

[深入理解css3中的flex-grow、flex-shrink、flex-basis](https://www.cnblogs.com/ghfjj/p/6529733.html)

[Solved by Flexbox](https://magic-akari.github.io/solved-by-flexbox/)

[http://static.vgee.cn/static/index.html](http://static.vgee.cn/static/index.html)