<template>
  <div id="home">
    <nav-bar class="home-nav">
      <template v-slot:center>
        <span class="center">购物街</span>
      </template>
    </nav-bar>
    <tab-controller
      :goodsType="['流行', '新款', '热卖']"
      @goodsSwitch="goodsSwitch"
      class="tab-contrl"
      v-show="isFixed"
      ref="tabControl1"
    />
    <scroll
      class="scroll-box"
      ref="scroll"
      @scrollPosition="homeScroll"
      @atBottom="loadMore"
      :probeType="3"
      :pullUpLoad="true"
    >
      <swiper :banner="banner" @swiperLoad="swiperLoad" />
      <recommend :recommend="recommend" @recommendLoad="recommendLoad" />
      <feature @featureLoad="featureLoad" />
      <tab-controller
        :goodsType="['流行', '新款', '热卖']"
        @goodsSwitch="goodsSwitch"
        ref="tabControl2"
      />
      <goods-list :goods="goodsTypeList" />
    </scroll>
    <to-top @click.native="topClick" v-show="isShowTop" />
  </div>
</template>

<script>
import NavBar from "components/common/navbar/NavBar";
import TabController from "components/content/tabcontroller/TabController";
import GoodsList from "components/content/goods/GoodsList";
import Scroll from "components/common/scroll/Scroll";

import Swiper from "views/home/childComponents/Swiper";
import Recommend from "views/home/childComponents/Recommend";
import Feature from "views/home/childComponents/Feature";

import { getHomeData, getGoodsList } from "network/home";

// 引入防抖函数
// import { debounce } from "common/utils";
// 引入混入mixin
import {
  imgFunction,
  toTopFn,
  SwitchGoods,
  FixedTop,
  GoodsCartData,
  activeOrNot,
} from "common/mixin";
import { POP, NEW, SELL } from "common/const";

export default {
  name: "",
  components: {
    NavBar,
    Swiper,
    Recommend,
    Feature,
    TabController,
    GoodsList,
    Scroll,
  },
  data() {
    return {
      banner: [],
      recommend: [],
      goods: {
        pop: { page: 0, list: [] },
        new: { page: 0, list: [] },
        sell: { page: 0, list: [] },
      },
      imgLoadCount: 0,
    };
  },
  mixins: [imgFunction, toTopFn, SwitchGoods, FixedTop, GoodsCartData, activeOrNot],
  methods: {
    // 网络请求相关
    // 请求banner recommend数据
    getHomeData() {
      getHomeData().then((res) => {
        // console.log(res);
        this.banner = res.data.banner.list;
        this.recommend = res.data.recommend.list;
      });
    },
    // 请求商品列表数据
    getGoodsList(type) {
      const currentPage = this.goods[type].page + 1;
      getGoodsList(type, currentPage).then((res) => {
        // console.log(res);
        this.goods[type].list.push(...res.data.list);
        // this.goods[type].page = res.data.page
        this.goods[type].page += 1;
        // 调用结束下拉加载更多，可进行下次上拉
        this.$refs.scroll.finishPullUp();
      });
    },
    // 事件相关
    // 点击切换类型
    // mixin

    // 回到顶部功能mixin

    // 是否显示到顶图标 mixin

    // 加载更多，传入当前的请求类型
    loadMore() {
      this.getGoodsList(this.currentType);
    },
    recommendLoad() {
      this.imgLoadCount += 1;
      return this.imageLoad();
    },
    swiperLoad() {
      this.imgLoadCount += 1;
      return this.imageLoad();
    },
    featureLoad() {
      this.imgLoadCount += 1;
      return this.imageLoad();
    },
    imageLoad() {
      if (this.imgLoadCount == 3) {
        this.offsetTop = this.$refs.tabControl2.$el.offsetTop;
      }
    },
  },
  created() {
    // 获取home banner,recommend数据
    this.getHomeData();
    // 获取home商品数据
    this.getGoodsList(POP);
    this.getGoodsList(NEW);
    this.getGoodsList(SELL);
  },
  mounted() {
    // 刷新dom,抽离到mixin.js 混入
  },
};
</script>

<style scoped>
#home {
  position: relative;
  background-color: #eee;
  height: 100vh;
}
.home-nav {
  position: relative;
  z-index: 999;
  width: 100%;
  background-color: var(--color-tint);
}
.center {
  font-size: 1.25rem;
  color: #fff;
}
.scroll-box {
  overflow: hidden;
  position: absolute;
  top: 44px;
  bottom: 49px;
  left: 0;
  right: 0;
  z-index: 1;
}
.tab-contrl {
  position: relative;
  left: 0;
  right: 0;
  z-index: 6;
}
</style>
