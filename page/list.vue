<template>
  <div class="tabArea">
    <pts-scroll-ajax ref="scrollAjax"
                     :topAjax="topAjax && !searchCondition && !startDate && !showDataNull"
                     @on-top-ajax="getFirstPage" :bottomAjax="bottomAjax && !showDataNull"
                     @on-bottom-ajax="getNextPage">
      <p class="totalOrder" v-if="dataList.length">{{$route.path == '/inside/inRepairTask'? '近一个月' : '本次查询'}}共<em style="font-weight: bold;">{{maxNum}}</em>单</p>
      <pts-task-item v-if="dataList.length" v-for="(item,i) in dataList" :key="item.reportNo"
                     :datas="item"  class="taskArea" :class="{pt20:i !== 0}"></pts-task-item>
      <!--<div v-if="showDataOver" class="list-data-over">没有数据啦</div>-->
      <!--` 第一次获取的数据为空时显示的页面 -->
      <div v-if="showDataNull" class="dataNullWrap">
        <div class="imgWrap"></div>
        <div class="dataNullText">没有搜到匹配的数据…</div>
      </div>
    </pts-scroll-ajax>
  </div>
</template>

<script>
  import toast from '../../../common/comComponent/toast'
  import ptsScrollAjax from '../../../common/comComponent/scrollAjax'
  import ptsTaskItem from  './item.vue'
  import axios from '../../../common/js/axiosConfig'
  import url from '../../../common/js/comConfig'

  export default {
    name: "inTaskList",
    props: {
      flagName: String,
      active: Boolean,
      tabChild: String,
      searchCondition: String,
      startDate: String,
      endDate: String,
      dealerCode: String
    },
    data () {
      return {
        topAjax: true,
        bottomAjax: false,
        dataList: [],
        showDataNull: false,
        page: 1,
        maxNum: 0, // 显示近30天有多少单
        // searchDays: 0 // 后台搜索了多少天
      }
    },
    methods: {
      getData (flag, topajax) {
        const _this = this
        if (flag) {
          this.dataList = [];
          this.showDataNull = false;
        }
        /*axios.post(url.getWebServiceUrls('pushRepairList'), {
         params: {
            "dealerCode": "string",
            "endDate": "string",
            "pageNo": 0,
            "pageSize": 0,
            "searchCondition": "string",
            "startDate": "string",
            "taskStatus": "string"
          }
         })*/
        axios.post(url.getWebServiceUrls('pushRepairList'), {
          "dealerCode" : _this.dealerCode || '', //网点搜索,默认全部网点
          "taskStatus": _this.tabChild || _this.flagName, // 查询类型
          "endDate": _this.endDate || undefined, // 搜索结束时间
          "searchCondition": _this.searchCondition || undefined, // 搜索车牌号or案件号
          "startDate": _this.startDate || undefined, //  搜索开始时间
          "pageNo": _this.page,
          "pageSize": 10
        }, {loading: !topajax})
          .then(res => {
            // console.table(res.data.data);
            let data = typeof res.data === 'string' ? JSON.parse(res.data) : res.data
            switch (data.code) {
              case 0:
                _this.maxNum = data.totalCount;
                if (data.data && data.data.length === 0) {
                  if (flag) {
                    if (_this.searchCondition || _this.startDate) {
                      _this.$emit('data-back', true)
                    } else {
                      _this.showDataNull = true
                      _this.bottomAjax = false
                    }
                  } else {
//                    _this.showDataOver = true
                    _this.bottomAjax = false
                  }
                  _this.$refs.scrollAjax && _this.$refs.scrollAjax.reset(flag);
                  return
                }
                // _this.searchDays = data.data[0] && data.data[0].searchDays || '一个';
                data.data.forEach(v => {
                  _this.dataList.push(v)
                });

                _this.bottomAjax = true
                if (data.pageNo * data.pageSize > data.totalCount) {
                  _this.bottomAjax = false
//                  _this.showDataOver = true
                }
                _this.$nextTick(function () {
                  (_this.searchCondition || _this.startDate) && this.$emit('data-back')
                  _this.$refs.scrollAjax && _this.$refs.scrollAjax.reset(flag)
                })
                break
              default:
                if (flag) {
                  if (_this.searchCondition || _this.startDate) {
                    _this.$emit('data-back', true) // 返回搜索页面让其显示搜索为空
                  } else {
                    _this.showDataNull = true
                  }
                }
                _this.$nextTick(function () {
                  _this.$refs.scrollAjax && _this.$refs.scrollAjax.reset(flag)
                })
                toast(data.msg)
                break
            }
          }).catch(err => {
          if (flag) {
            if (_this.searchCondition || _this.startDate) {
              _this.$emit('data-back', true) // 返回搜索页面让其显示搜索为空
            } else {
              _this.showDataNull = true
            }
            _this.$refs.scrollAjax && _this.$refs.scrollAjax.reset(flag)
          }
        })
      },

      getNextPage () {
        this.page++;
        this.getData();
      },
      getFirstPage () {
        this.page = 1;
        this.getData(true, true);
        if (this.flagName === '00') {
          this.$emit('on-offRedDot');
        }
      }
    },
    mounted () {
      if (this.active) {
        this.getData(true)
      }
    },
    components: {
      ptsScrollAjax,
      ptsTaskItem
    },
    watch: {
      active (to, form) {
        if (to && this.dataList.length === 0) {
          this.page = 1;
          this.getData(true)
        }
      },
      tabChild (to, form) {
        this.page = 1;
        this.getData(true)
      },
      dealerCode(newVal,oldVal){
        if (newVal === oldVal) return
        this.getData(true)
      }
    },
  }
</script>

<style lang="less" scoped>
  .tabArea {
    width: 100%;
    height: 100%;
    box-sizing: border-box;
  }

</style>
