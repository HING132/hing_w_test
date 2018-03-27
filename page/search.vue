<template>
  <section class="box">
    <div class="b-b titleWrap" style="position: fixed;">
      <a href="javascript:;" otype="button" otitle="用户头像" class="navigation_arrow fl" @click="goBack"></a>
      <div class="entryBox por seachBox">
        <input type="text" :placeholder="placeholder" v-model="searchValue" ref="searchInput"
               @focus="changeSearchPageContent" @blur="blurSearch" @keyup.enter="blurSearch">
        <div class="entryBox_right c clear" v-if="searchValue" @click="searchValue=''">
          <img :src="offImg"/>
        </div>
      </div>
    </div>
    <div v-if="showsearchArea && taskList">
      <group>
        <datetime title="开始时间"
                  format="YYYY-MM-DD"
                  v-model="startTime"
                  :start-date="startDateTimeOption.startDate"
                  :end-date="startDateTimeOption.endDate"
                  @on-change="getTime"
        ></datetime>
        <datetime
          title="结束时间"
          format="YYYY-MM-DD"
          v-model="endTime"
          :start-date="endDateTimeOption.startDate"
          :end-date="endDateTimeOption.endDate"
          @on-change="getTime"
        ></datetime>
      </group>
      <div class="seachBtn">
        <a href="javascript:;" title="确认搜索" otitle="确认搜索" otype="button" @click="search">确认搜索</a>
      </div>
    </div>
    <div v-if="searchEventArea" class="searchWrap">
      <div class="seachFail" v-if="showSearchNull">
        <img :src="scarchNullImg">
        <p>没有搜到匹配结果，请重试…</p>
      </div>
      <!-- 内部端推修任务列表页 -->
      <pts-task :searchCondition="trimSearchValue" :startDate="startTime" :endDate="endTime" flagName="07" :active="true"
                @data-back="databack" v-if="taskList && !showSearchNull">
      </pts-task>
    </div>
  </section>
</template>

<script>
  import {Group, Datetime} from 'vux'
  import toast from '../../../common/comComponent/toast'

  export default {
    name: "search",
    data () {
      return {
        startTime: '',
        endTime: '',
        startDateTimeOption: {
          startDate: '',
          endDate: ''
        },
        endDateTimeOption: {
          startDate: '',
          endDate: ''
        },
        offImg: require('../../../common/images/clase.png'),
        loadingImg: require('../../../common/images/loading.png'),
        scarchNullImg: require('../../../common/images/failIcon.png'),
        searchValue: '',
        showsearchArea: true,
        showloading: true,
        showSearchNull: false,
        placeholder: '请输入搜索的内容（案件号、车牌)',
        taskList: false,
        toastMsg: '',
        searchEventArea: false,
        flag: 0,
        maidianId: ''
      }
    },
    methods: {
      goBack () {
        this.flag = 1;
        window.history.go(-1)
      },

      getTime () {
        this.endDateTimeOption.startDate = this.startTime;
        if (new Date(this.startTime) > new Date(this.endTime)) {
          toast('开始时间不能晚于结束时间');
          return false
        }
        window.eventAnalytics(this.maidianId, this.maidianId + '点击时间选择器');
        return true
      },
      /*
       * @info input框失去焦点时 触发 目前仅在订单列表进入时显示
       * */
      blurSearch () {
        if (this.taskList) {
          return
        }
        this.$refs.searchInput.blur();
        this.search();
      },
      /*
       * @info 搜索函数, 让显示内容的区域显示, 此时用户输入的值通过组件的 searchCondition props 传入组件内 然后让组件自己发出ajax请求请求数据,
       * */
      search () {
        if (!this.getTime()) {
          return false
        }
        if (this.searchValue.trim().length > 0 || (this.startTime && this.endTime)) {
          this.searchEventArea = true;
          this.showsearchArea = false;
          this.$nextTick(function () {
            this.showSearchNull = false;
          });
          return
        }
        setTimeout(() => {
          if (this.flag !== 1) {
            let msg = this.taskList ? '或选择时间' : '';
            toast(this.toastMsg + msg);
          }
        }, 100)
      },
      /* 当搜索框获取焦点的时候, 切换页面显示的状态, 同时会清除已搜索的逻辑 */
      changeSearchPageContent () {
        this.showsearchArea = true;
        this.searchEventArea = false;
        window.eventAnalytics(this.maidianId, this.maidianId + '点击搜索框');
      },
      /*
       * @info 监听子组件是否获取到数据 由子组件触发
       * @params flag {boolean} 当为真时, 说明没有收到数据, 查询结果为空
       * */
      databack (flag) {
        if (flag) {
          this.showloading = false
          this.showSearchNull = true
          this.showsearchArea = false
        } else {
          this.showloading = false
        }
      },
      init () {
        /* 根据url中的标记 flag 的value 来判断是从哪里进来的搜索页面*/
        const _this = this;
        this.taskList = false;
        this.searchValue = '';
        this.searchEventArea = false;
        const obj = {
          inRepairTaskList () {
            _this.taskList = true;
            _this.placeholder = '请输入搜索的内容(案件号、车牌)';
            _this.toastMsg = '请输入案件号/车牌';
            _this.maidianId = '推修任务';
          },
        }
        obj[this.$route.query.flag]()
      }
    },
    mounted () {
      this.init();
      const dateObj = new Date();
      const endYear = dateObj.getFullYear();
      const endMonth = dateObj.getMonth() + 1;
      const endDate = dateObj.getDate();
      const today = `${endYear}-${endMonth >= 10 ? endMonth : '0' + endMonth}-${endDate >= 10 ? endDate : '0' + endDate}`
      const startDateObj = new Date();
      // console.log('这是today'+today)
      // console.log('这是endYear'+endYear)
      // console.log('这是endMonth'+endMonth)
      // console.log(endDate)

      // const day = new Date(endYear,endMonth,0);
      // const dayCount = day.getDate(); //当前月的总天数
      startDateObj.setDate(endDate); // 显示近1个月的搜索范围

      const startYear = startDateObj.getFullYear();
      const startMonth = startDateObj.getMonth();// 上面的endMonth已经加一 不需要再加一
      const endDateNum = endDate - 1;
      this.startDateTimeOption.startDate = this.endDateTimeOption.startDate = `${startYear}-${startMonth >= 10 ? startMonth : '0' + startMonth}-${endDateNum >= 10 ? endDateNum : '0'+ endDateNum}`;
      this.startDateTimeOption.endDate = this.endDateTimeOption.endDate = today;
      this.$nextTick(function () {
        this.startTime = this.endTime = today;
      });
    },
    computed:{
      //去除用户输入的空格
      trimSearchValue:function () {
        return this.searchValue.replace(/\s/g,"")
      }
    },
    components: {
      Group,
      Datetime,
      ptsTask: resolve => require.ensure([], () => resolve(require('./list.vue')), 'inSearchTaskList'),
    },
    beforeRouteLeave (to, from, next) {
      console.log(`进入页面${to.path}保留页面`);
      if (to.path === '/inside/inRepairTask') {
        console.log(`进入页面${to.path}销毁页面`);
        this.$destroy();
      }
      next()
    },
    filters:{
      trim(value){
        return value.replace(/\s/g,"")
      }
    }
  }
</script>

<style lang="less">
  @import "../../../common/css/1px.less";

  .loadImg {
    animation: loading 2s linear infinite;
  }

  .searchWrap {
    height: 100%;
    border-top: 1px solid transparent;
    box-sizing: border-box;
  }
  .totalOrder{
    height: .6rem;
    line-height: .6rem;
    font-size: .24rem;
    color: #888888;
  }
  .b-b {
    .pts-1px-b(#D8D8D8);
  }

  @keyframes loading {
    form {
      transform: rotate(0deg);
    }
    to {
      transform: rotate(360deg);
    }
  }
</style>
