### 三角形

<vuep template="#example"></vuep>

<script v-pre type="text/x-template" id="example">
<style>
     .container{
         display:flex;
         justify-content:center;
         align-items:center;
         flex-direction:column;
     }
     .box{
        box-sizing:content-box;
        background-color:red;
        border: 60px solid #80cbae;;
    }
  </style>
<template>
  <div class="container">
    <div class="box" :style="{
       width: width+'px',
       height: height+'px',
       borderTopLeftRadius:border_top_left_radius_l+'px '+border_top_left_radius_r+'px',
       borderTopRightRadius:border_top_right_radius_l+'px '+border_top_right_radius_r+'px',
       borderBottomRightRadius:border_bottom_right_radius_l+'px '+border_bottom_right_radius_r+'px',
       borderBottomLeftRadius:border_bottom_left_radius_l+'px '+border_bottom_left_radius_r+'px',
    }"
    >
    </div><br>
    <div>width:<input v-model.number=width type="number" /></div><br>
    <div>height:<input v-model.number=height type="number" /></div><br>
    <div>borderTopLeftRadiusL:<input v-model.number=border_top_left_radius_l type="number" /></div><br>
    <div>borderTopLeftRadiusR:<input v-model.number=border_top_left_radius_r type="number" /></div><br>
    <div>borderTopRightRadiusL:<input v-model.number=border_top_right_radius_l type="number" /></div><br>
    <div>borderTopRightRadiusR:<input v-model.number=border_top_right_radius_r type="number" /></div><br>
    <div>borderBbottomRightRadiusL:<input v-model.number=border_bottom_right_radius_l type="number" /></div><br>
    <div>borderBbottomRightRadiusR:<input v-model.number=border_bottom_right_radius_r type="number" /></div><br>
    <div>borderBottomLeftRadiusL:<input v-model.number=border_bottom_left_radius_l type="number" /></div><br>
    <div>borderBottomLeftRadiusR:<input v-model.number=border_bottom_left_radius_r type="number" /></div><br>
  </div>
</template>
  <script>
    module.exports = {
      data: function () {
        return { 
            width:100,
            height:100,
            border_top_left_radius_l:0,
            border_top_left_radius_r:0,
            border_top_right_radius_l:0,
            border_top_right_radius_r:0,
            border_bottom_right_radius_l:0,
            border_bottom_right_radius_r:0,
            border_bottom_left_radius_l:0,
            border_bottom_left_radius_r:0
        }
      }
    }
</script>
</script>