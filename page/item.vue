<template>
  <section>
    <dl>
      <dt v-if="datas.taskStatus === '00'" class="f_c_fe8230">新任务</dt>
      <dt v-if="datas.taskStatus === '02'">待确认</dt>
      <dt v-if="datas.taskStatus === '04'" style="border-color:#2ab768; color:#2ab768;">成功</dt>
      <dt v-if="datas.taskStatus === '05'" style="border-color:#ff6666; color:#ff6666;">失败</dt>
      <dd>{{datas.pushDate | extractStr}}</dd>
    </dl>
    <div class="taskBox" :class="{'fontcolor333': datas.taskStatus === '03' }" @click="goInfo">
      <dl>
        <dt>车牌号</dt>
        <dd >{{datas.carMark | vehicleLicenceCodeHidden}}</dd>
      </dl>
      <dl>
        <dt>出险时间</dt>
        <dd>{{datas.accidentTime}}</dd>
      </dl>
      <dl>
        <dt>出险地点</dt>
        <dd>{{datas.accidentSite | joinStr}}</dd>
      </dl>
      <dl>
        <dt>是否大案</dt>
        <dd :class="{fcFF4:datas.majorCase==='Y'}">{{datas.majorCase==='Y' ? '是' : '否'}}{{datas.isThirdType=='Y' ? '(三者)' : ''}}</dd>
      </dl>
      <dl>
        <dt>维修网点</dt>
        <dd>{{datas.dealerName}}</dd>
      </dl>
      <!--<a href="javascript:;" otype="button" otitle="拨打电话" class="btnState btnStateTell" @click.stop.prevent="goShowBanner">案件图片</a>-->
    </div>
  </section>
</template>

<script>
  import '../../../common/filters/convertAmount'
  export default {
    name: "taskItem",
    props: {
      datas: Object
    },
    data () {
      return {
        type: 'new'
      }
    },
    methods: {
      goInfo () {
        this.$router.push({
          path: '/inSide/inRepairTaskInfo',
          query: {
            carMark: this.datas.carMark,
            reportId: this.datas.reportNo,
            taskId: this.datas.taskId,
            taskStatus:this.datas.taskStatus,
            vehicleFrameNo:this.datas.vehicleFrameNo
          }
        });
        window.eventAnalytics('内部推修任务', '进入推修详情');
      },
      goShowBanner(){
        this.$router.push({path:'/inside/inRepairShowImage'});
        window.eventAnalytics('内部推修任务', '进入案件照片');
      }
    },
    filters: {
      vehicleLicenceCodeHidden (value) {
        if (!value) return value
        if (value.indexOf('*') > -1) {
          return '*_*'
        } else {
          return value
        }
        // if(value.indexOf('-') > -1){
        //   return value
        // }else{
        //   return value.substring(0,2)+'-'+value.substring(2)
        // }
      },
      extractStr(value){ //字符串截取
        if (!value) return value
        return value.substring(0,value.indexOf(' '))
      },
      joinStr(value){
        if(!value) return value
        return value.split('-').join('')
      }
    }
  }
</script>

<style lang="less" scoped>

</style>
