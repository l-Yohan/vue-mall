<template>
  <div id="home">
    <nav-bar class="home-nav"><div slot="center">购物街</div></nav-bar>
    <tab-control :titles="['流行','新款','精选']"
                   @tabClick="tabClick"
                   ref="tabControl1"
                   class="tab-control" 
                   v-show="isTabFixed"></tab-control>
    <scroll class="content" ref="scroll" :probe-type="3" @scroll="contentScroll" :pull-up-load="true" @pullingUp="loadMore">
      <home-swiper :banners="banners" @swiperImageLoad="swiperImageLoad"></home-swiper>
      <recommend-view :recommends="recommends"/>
      <feature-view></feature-view>
      <tab-control :titles="['流行','新款','精选']"
                   @tabClick="tabClick"
                   ref="tabControl2"></tab-control>
      <goods-list :goods="showGoods"></goods-list>
    </scroll>
    <back-top @click.native="backClick" v-show="isShowBackTop"></back-top>
  </div>
</template>

<script>
  import HomeSwiper from './childComps/HomeSwiper';
  import RecommendView from './childComps/RecommendView';
  import FeatureView from './childComps/FeatureView';

  import NavBar from 'components/common/navbar/NavBar';
  import TabControl from 'components/common/tabControl/TabControl';
  import GoodsList from 'components/content/goods/GoodsList'
  import Scroll from 'components/common/scroll/Scroll'
  import BackTop from 'components/content/backTop/BackTop'
  import {getHomeMultiData, getHomeGoods} from 'network/home';
  
  import {debounce} from 'common/utils.js'
  
  export default {
    name: 'Home',
    components: {
      NavBar,
      HomeSwiper,
      RecommendView,
      FeatureView,
      TabControl,
      GoodsList,
      Scroll,
      BackTop,
    },
    data() {
      return{
        banners: [],
        recommends: [],
        goods: {
          'pop': {page: 0, list: []},
          'new': {page: 0, list: []},
          'sell': {page: 0, list: []},
        },
        currentType: 'pop',
        isShowBackTop: false,
        tabOffsetTop: 0,
        isTabFixed: false,
      }
    },
    computed: {
      showGoods() {
        return this.goods[this.currentType].list
      },
      destroyed() {
        console.log('likai');
      },
      activated() {
        this.$refs.scroll.scrollTo(0, this.saveY, 0)
        this.$refs.scroll.refresh()
      },
      deactivated() {
        this.saveY = this.$refs.scroll.getScrollY()
      }
    },
    created() {
      // getHomeMultidata().then(res => {
      //     this.banners = res.data.banner;
      //     this.recommends = res.data.recommend;
      //  })
      this._getHomeMultiData()
      this.getHomeGoods('pop')
      this.getHomeGoods('new')
      this.getHomeGoods('sell')
      // getHomeGoods('pop',1).then(res => {
      //   console.log(res.data.list);
      // })
    },
    mounted() {
      const refresh = debounce(this.$refs.scroll.refresh, 50)
      // 监听item中图片加载完成
      this.$bus.$on('itemImageLoad', () => {
        // console.log('----');
        refresh()
      })
    },
    methods: {
      
      // 事件监听
      tabClick(index) {
        switch(index) {
          case 0:
            this.currentType = 'pop'
            break
          case 1: 
            this.currentType = 'new'
            break
          case 2:
            this.currentType = 'sell'
            break
        }
        this.$refs.tabControl1.currentIndex = index;
        this.$refs.tabControl2.currentIndex = index
      },
      backClick() {
        this.$refs.scroll.scrollTo(0, 0)
      },
      contentScroll(position) {
        // 判断BackTop是否显示
        this.isShowBackTop = -position.y > 1000

        // 绝定tabControl是否吸顶
        this.isTabFixed = (-position.y) > this.tabOffsetTop
      },
      loadMore() {
        this.getHomeGoods(this.currentType)
        this.$refs.scroll.scroll.refresh( )
      },
      swiperImageLoad() {
          this.tabOffsetTop = this.$refs.tabControl2.$el.offsetTop
      },

      // 网络请求
      _getHomeMultiData() {
        getHomeMultiData().then(res => {
          // console.log(res.data.banner.list);
          this.banners = res.data.banner.list
          this.recommends = res.data.recommend.list
       })
      },
      getHomeGoods(type) {
        const page = this.goods[type].page + 1
        getHomeGoods(type, page).then(res => {
          this.goods[type].list.push(...res.data.list)
          this.goods[type].page += 1
          this.$refs.scroll.finishPullUp()
        })
      }
    }
  }
</script>

<style scoped>
  #home {
    /* padding-top: 44px; */
    height: 100vh;
    position: relative;
  }
  .home-nav {
    background-color: var(--color-tint);
    color: #fff;
    /* position: fixed;
    top: 0;
    right: 0;
    left: 0;
    z-index: 9; */
  }
  /* .home-swiper {
    padding-top: 44px;
  } */
  
  .content {
    overflow: hidden;
    position: absolute;
    top: 44px;
    bottom: 49px;
    left: 0;
    right: 0;
  }
  /* .content {
    height: calc(100% - 93px);
    overflow: hidden;
    margin-top: 44px;
  } */
  .tab-control {
    position: relative;
    z-index: 9;
    background-color: #fff;
  }
  
</style>