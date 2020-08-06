<template>
  <div id="home">
    <NavBar class="home-nav">
      <div slot="center">购物街</div>
    </NavBar>
    <Scroll ref="scroll" :probeType="3" :pullUpLoad="true" @scroll="contentScroll" @pullingUp="loadMore">
      <HomeSwiper :banners='banners'/>
      <RcommendView :recommends='recommends'/>
      <FeatureView/>
      <TabControl class="tab-control" :titles="['流行','新款','精选']" @tabClick="tabClick"/>
      <GoodsList :goods="goods[currentType].list"/>
    </Scroll>
    <BackTop class="back-top" @click.native="backClick" v-show="isShowBackTop"/>
  </div>
</template>

<script>
import NavBar from 'components/common/navbar/NavBar'
import TabControl from 'components/content/tabControl/TabControl'
import GoodsList from 'components/content/goods/GoodsList'
import Scroll from 'components/common/scroll/Scroll'
import BackTop from 'components/content/backTop/BackTop'

import HomeSwiper from './childcomps/HomeSwiper'
import RcommendView from './childcomps/RecommendView'
import FeatureView from './childcomps/FeatureView'


import {getHomeMultidata,getHomeGoods} from 'network/home'

export default {
  name:'Home',
  components:{
    NavBar,
    TabControl,
    HomeSwiper,
    RcommendView,
    FeatureView,
    GoodsList,
    Scroll,
    BackTop
  },
  data() {
    return {
      banners:[],
      recommends:[],
      goods:{
        'pop':{page:0,list:[]},
        'new':{page:0,list:[]},
        'sell':{page:0,list:[]}
      },
      currentType:'pop',
      isShowBackTop:false
    }
  },
  created() {
    //1.请求多个数据
    this.getHomeMultidata(),
    
    this.getHomeGoods('pop'),
    this.getHomeGoods('new'),
    this.getHomeGoods('sell')
  },
  methods: {
    /**
     * 网络请求先关方法
     */
    getHomeMultidata(){
      getHomeMultidata().then(res => {
        this.banners = res.data.banner.list;
        this.recommends = res.data.recommend.list;
      })
    },
    getHomeGoods(type){
      const page = this.goods[type].page+1
      getHomeGoods(type,page).then(res => {
        this.goods[type].list.push(...res.data.list)
        this.goods[type].page += 1
      })
      this.$refs.scroll.finishPullUp()
    },

    /**
     * 事件监听先关方法
     */
    tabClick(index){
      switch(index){
        case 0:
          this.currentType='pop'
          break
        case 1:
          this.currentType='new'
          break
        case 2:
          this.currentType='sell'
          break
      }
    },
    backClick(){
      this.$refs.scroll.scrollTo(0,0)
    },
    contentScroll(position){
      // console.log(position);
      this.isShowBackTop = (-position.y) > 800
    },
    loadMore(){
      this.getHomeGoods(this.currentType)
    }
  }
}
</script>

<style scope>
  #home{
    position: relative;
    height: 100vh;
  }
  .home-nav{
    background-color: var(--color-tint);
    color: white;

    position: fixed;
    left: 0;
    right: 0;
    top: 0;
    z-index: 9;
  }
  .tab-control{
    position: sticky;
    top:44px;
    z-index: 9;
  }
  .wrapper{
    overflow: hidden;
    position: absolute;
    top: 44px;
    bottom: 49px;
    left: 0;
    right: 0;
  }
  .back-top img{
    width: 43px;
    height: 43px;
  }
</style>