<template>
  <div class="tab-control">
    <div
      v-for="(item,index) in titles"
      :key="item"
      :class="{active:index === curIndex}"
      @click="tabClick(index)"
      class="tab-control-item"
    >
      <span>{{item}}</span>
    </div>
  </div>
</template>

<script>
export default {
  name: "TabControl",
  props: {
    titles: {
      type: Array,
      default() {
        return [];
      }
    }
  },
  data() {
    return {
      curIndex: 0
    };
  },
  methods:{
    tabClick(index){
      this.curIndex = index;
      this.$emit('tabClick',index)
    }
  }
};
</script>
<style scoped>
.tab-control {
  display: flex;
  text-align: center;
  height: 40px;
  line-height: 40px;
  font-size: 15px;
  background-color: #fff
}
.tab-control-item {
  flex: 1;
}

.active {
  color: var(--color-high-text);
}
.active span {
  border-bottom: 3px solid var(--color-tint);
  padding: 5px;
}
</style>