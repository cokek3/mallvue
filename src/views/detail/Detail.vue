<template>
  <div id="detail">
    <detail-nav-bar class="detail-nav" @titleClick="titleClick" ref="nav"></detail-nav-bar>
    <scroll class="content" ref="scroll" @scroll="contentScroll" :probe-type="3">
      <detail-swiper :top-images="topImages"></detail-swiper>
      <detail-base-info :goods="goods"></detail-base-info>
      <detail-shop-info :shop="shop"></detail-shop-info>
      <!-- <detail-goods-info :detail-info='detailInfo' @imgLoad='imgLoad'></detail-goods-info> -->
      <detail-param-info :param-info="paramInfo" ref="params"></detail-param-info>
      <detail-comment-info :comment-info="commentInfo" ref="comment"></detail-comment-info>
      <goods-list :goods="recommends" ref="recommend"></goods-list>
    </scroll>
    <detail-bottom-bar @addCart="addToCart"></detail-bottom-bar>
    <back-top @click.native="backClick" v-show="isShowBackTop"></back-top>
  </div>
</template>

<script>
import Scroll from "components/common/scroll/Scroll";

import DetailNavBar from "./childComponents/DetailNavBar";
import DetailSwiper from "./childComponents/DetailSwiper";
import DetailBaseInfo from "./childComponents/DetailBaseInfo";
import DetailShopInfo from "./childComponents/DetailShopInfo";
import DetailGoodsInfo from "./childComponents/DetailGoodsInfo";
import DetailParamInfo from "./childComponents/DetailParamInfo";
import DetailCommentInfo from "./childComponents/DetailCommentInfo";
import DetailBottomBar from "./childComponents/DetailBottomBar";

import { debounce } from "common/utils";
import GoodsList from "components/content/goods/GoodsList";
import { mapActions } from "vuex";

import {
  getDetail,
  Goods,
  Shop,
  GoodsParam,
  getRecommend
} from "network/detail";

import { itemListenerMixin, backTopMixin } from "common/mixin";

export default {
  name: "Detail",
  components: {
    DetailNavBar,
    DetailSwiper,
    DetailBaseInfo,
    DetailShopInfo,
    DetailGoodsInfo,
    DetailParamInfo,
    DetailCommentInfo,
    DetailBottomBar,
    GoodsList,
    Scroll
  },
  data() {
    return {
      iid: null,
      topImages: [],
      goods: {},
      shop: {},
      detailInfo: {},
      paramInfo: {},
      commentInfo: {},
      recommends: [],
      themeTopYs: [],
      curIndex: 0
    };
  },
  mixins: [itemListenerMixin, backTopMixin],
  methods: {
    ...mapActions(["addCart"]),
    imgLoad() {
      this.$refs.scroll.refresh();
    },
    titleClick(index) {
      this.$refs.scroll.scrollTo(0, -this.themeTopYs[index], 100);
    },
    contentScroll(position) {
      const posY = -position.y;
      let length = this.themeTopYs.length;
      for (let i = 0; i < length - 1; i++) {
        if (
          this.curIndex !== i &&
          (posY >= this.themeTopYs[i] && posY < this.themeTopYs[i + 1])
        ) {
          this.curIndex = i;
          this.$refs.nav.curIndex = this.curIndex;
        }
      }
      this.listenShowBackTop(position);
    },
    addToCart() {
      // 1 获取购物车需要展示的信息
      const product = {};
      product.image = this.topImages[0];
      product.title = this.goods.title;
      product.desc = this.goods.desc;
      product.price = this.goods.realPrice;
      product.iid = this.iid;

      // 2 将商品添加到购物车里
      this.addCart(product).then(res => {
       this.$toast.show(res,2000)
      });
      // this.$store.dispatch("addCart", product).then(res=>{
      //   console.log(res);
      // });
    }
  },
  created() {
    // 1 保存传入的iid
    this.iid = this.$route.params.iid;

    // 2 根据iidq请求详情数据
    getDetail(this.iid).then(res => {
      // 1 获取顶部的图片轮播数据
      const data = res.result;
      this.topImages = data.itemInfo.topImages;

      // 2 获取商品信息
      this.goods = new Goods(
        data.itemInfo,
        data.columns,
        data.shopInfo.services
      );
      // 3 创建店铺信息的对象
      this.shop = new Shop(data.shopInfo);

      // 4 保存商品的详情数据
      this.detailInfo = data.detailInfo;

      // 5 获取参数的信息
      this.paramInfo = new GoodsParam(
        data.itemParams.info,
        data.itemParams.rule
      );
      // 6 获取评价
      if (data.rate.list) {
        this.commentInfo = data.rate.list[0];
      }

      this.$nextTick(() => {
        this.themeTopYs.push(0);
        this.themeTopYs.push(this.$refs.params.$el.offsetTop);
        this.themeTopYs.push(this.$refs.comment.$el.offsetTop);
        this.themeTopYs.push(this.$refs.recommend.$el.offsetTop);
        this.themeTopYs.push(Number.MAX_VALUE);
      });
    });

    // 3 请求推荐数据
    getRecommend().then(res => {
      this.recommends = res.data.list;
    });
  },
  destroyed() {
    this.$bus.$off("itemImageLoad", this.itemImgListener);
  }
};
</script>
<style scoped>
#detail {
  position: relative;
  height: 100vh;
  z-index: 9;
  background-color: #fff;
}
.detail-nav {
  position: relative;
  z-index: 9;
  background-color: #fff;
}
.content {
  height: calc(100% - 44px - 49px);
}
</style>