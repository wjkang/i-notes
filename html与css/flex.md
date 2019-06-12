
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