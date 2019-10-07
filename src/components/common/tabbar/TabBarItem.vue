<template>
  <div class="tab-bar-item" @click="changge">
    <div v-if='!isActive'><slot name="item-icon"></slot></div>
    <div v-else><slot name="item-icon-active"></slot></div>
    <div :style="activeColor"><slot name="item-text"></slot></div>
  </div>
</template>

<script>
export default {
  name: "TabBarItem",
  data(){
    return{
      // isActive:true,
    }
  },
  props:{
    path:String,
    colorStyle:{
      type:String,
      default:'red'
    }
  },
  computed:{
    isActive(){
      return this.$route.path.indexOf(this.path) !== -1
    },
    activeColor(){
      return this.isActive? {color:this.colorStyle}:{}
    }
  },
  methods:{
    changge(){
     this.$router.replace(this.path)
    }
  }
};
</script>
<style>
.tab-bar-item {
  flex: 1;
  height: 49px;
  text-align: center;
}
.tab-bar-item img{
  width: 24px;
  height: 24px;
  margin-top: 3px;
  vertical-align:middle;
}
</style>