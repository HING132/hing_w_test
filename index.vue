<template>
<div class="box">
  <pts-header leftFlag @on-left="goMenu" :titleText="titleText" :showRight="showRight">
    <div slot="center" class="textBanner">
      <text-scroll v-model="chooseName"></text-scroll>
    </div>
    <a slot="right" class="nav_search_icon fr" @click.stop.prevent="gotoSearch"></a>
  </pts-header>
  <div class="wrap insideXubaoWrap insideVisuWrap">
    <div class="mainWrap">
      <pts-tab :titleList="titleList" v-model="index" ref="tabElem" @on-child-change="tabChildChange" @on-change="tabChange">
        <pts-tab-item>
          <!--全部-->
          <pts-task-list flagName="07" :active="index===0" :dealerCode="chooseName"></pts-task-list>
        </pts-tab-item>
        <pts-tab-item>
          <!--新任务-->
          <pts-task-list flagName="00" @on-offRedDot="offRedDot" :active="index===1" :dealerCode="chooseName"></pts-task-list>
        </pts-tab-item>
        <pts-tab-item>
          <!--待确认-->
          <pts-task-list flagName="02" :active="index===2" :dealerCode="chooseName"></pts-task-list>
        </pts-tab-item>
        <pts-tab-item>
          <!--已完成-->
          <pts-task-list flagName="06" :active="index===3" :tabChild="tabChild" :dealerCode="chooseName"></pts-task-list>
        </pts-tab-item>
      </pts-tab>
    </div>
  </div>
</div>
</template>

<script>
  import ptsTab from '../../common/comComponent/tab'
  import textScroll from '../../common/comComponent/textScroll'

  //  import ptsTaskList from './page/list.vue'
  export default {
    name: "inRepairTask",
    data () {
      return {
        titleText:' ',
        chooseName:'',
        showRight:true,
        titleList: [
          {
            title: '全部'
          },
          {
            title: '新任务',
            isRed: false
          },
          {
            title: '待确认'
          },
          {
            title: '已完成',
            childs: [
              {title: '已完成', flag: '06'},
              {title: '成功', flag: '04'},
              {title: '失败', flag: '05'},
              {title: '失效', flag: '03'}
            ]
          }
        ],
        index: 0,
        tabChild: '06'
      }
    },
    components: {
      ptsTab,
      textScroll,
      ptsTabItem: ptsTab.Item,
      ptsTaskList: resolve => require.ensure([], () => resolve(require('./page/list.vue')), 'SearchTaskList')
    },
    methods: {
      tabChildChange (item) {
        this.tabChild = item.flag
      },
      goMenu () {
        Native.requestHybrid({
          tagname: 'backHome'
        })
      },
      offRedDot () {
        this.titleList[1].isRed = false
      },
      tabChange (item) {
        window.eventAnalytics('内部推修任务', 'tab切换', {tabName: item.title});
      },
      gotoSearch(){
        this.$router.push({path:'/inside/inSearch',query:{flag:'inRepairTaskList'}})
        window.eventAnalytics('内部推修任务', '进入搜索页');
      }
    },
    created () {
      const _this = this
      window.setRedDot = function () {
        _this.titleList[1].isRed = true
      }
    },
    watch: {
      index (to) {
        if (to === 1) {
          this.offRedDot();
        }
      }
    }
  }
</script>

<style lang="less" scoped>
    .nav_search_icon {
      display: inline-block;
      width: .48rem;
      height: .48rem;
      background: url("../../common/images/filter_icon@3x.png") no-repeat;
      margin-right: .31rem;
      margin-top: -0.65rem;
      background-size: 100%;
    };
</style>
