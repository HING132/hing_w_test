<template>
  <div class="box">
    <pts-header titleText="推修详情" leftFlag @on-left="goBack" :showRight="true">
      <a href="javascript:;" otype="button" otitle="分享按钮" class="nav_share_icon fr" slot="right" @click="shareCase"></a>
    </pts-header>
    <div class="wrap claim insideVisuDetailsWrap">
      <div class="mainWrap pb20" v-if="data.caseStatus && !showDataNull">
        <div class="taskArea reportMsgArea">
          <dl>
            <dt>出险信息</dt>
            <dd class="f_c_fe8230" v-if="data.taskStatus  === '00'">新任务</dd>
            <dd v-else-if="data.caseStatus  === '02'">待确认</dd>
          </dl>
          <div class="taskBox" style="margin-top: -0.08rem">
            <dl>
              <dt>出险原因</dt>
              <dd>{{data.accidentSituation}}</dd>
            </dl>
            <dl>
              <dt>出险地点</dt>
              <dd>{{data.accidentSite}}</dd>
            </dl>
            <dl>
              <dt>报案时间</dt>
              <dd>{{data.accidentTime}}</dd>
            </dl>
            <a href="javascript:;" otype="button" otitle="案件图片" class="btnState btnStateTell" :class="{active:isCaseImg}" @click.stop.prevent="goShowBanner">案件图片</a>
          </div>
        </div>
        <div class="taskArea detailArea">
          <dl>
            <dt>案件信息</dt>
          </dl>
          <div class="taskBox" style="margin-top: -0.08rem">
            <dl>
              <dt>案件号</dt>
              <dd>{{data.reportNo | fourSpace}}</dd>
            </dl>
            <dl>
              <dt>车牌</dt>
              <dd>{{data.carMark | vehicleLicenceCodeHidden}}</dd>
            </dl>
            <!--先屏蔽此字段-->
            <!--<dl>-->
              <!--<dt>车型</dt>-->
              <!--<dd>{{data.carType}}</dd>-->
            <!--</dl>-->
            <dl>
              <dt>客户类型</dt>
              <dd v-if="data.clientType == 0">本店客户</dd>
              <dd v-else-if="data.clientType == 1">非本店客户</dd>
            </dl>
            <dl>
              <dt>案件状态</dt>
              <dd v-if="data.caseStatus == 1">单方事故</dd>
              <dd v-else-if="data.caseStatus == 2">双方事故</dd>
            </dl>
            <dl>
              <dt>是否大案</dt>
              <dd v-if="data.majorCase==='Y'">是</dd>
              <dd v-else>否</dd>
            </dl>
            <dl>
              <dt>承保险别</dt>
              <dd class="linh50">
                <span v-if="data.planList[0]">{{data.planList[0].dutyName}}<br></span>
                <span v-if="data.planList[1]">{{data.planList[1].dutyName}}<br></span>
                <span v-if="data.planList[2]">{{data.planList[2].dutyName}}<br></span>
                <a href="javascript:;" v-if="data.planList.length > 0" otype="button" otitle="点击查看详情" class="checkDetail"
                   @click.prent.stop="showInsureMsgEvent">点击查看详情</a>
              </dd>
            </dl>
          </div>
        </div>
        <div class="taskArea detailArea">
          <dl>
            <dt>报案人信息</dt>
          </dl>
          <div class="taskBox" style="margin-top: -0.08rem">
            <dl>
              <dt>报案人</dt>
              <dd>{{data.reportName}}</dd>
            </dl>
            <dl>
              <dt>报案时间</dt>
              <dd>{{data.reportDate}}</dd>
            </dl>
          </div>
        </div>
        <div class="taskArea detailArea">
          <dl>
            <dt>查勘信息</dt>
          </dl>
          <div class="taskBox" style="margin-top: -0.08rem">
            <dl>
              <dt>查勘人</dt>
              <dd>{{data.surveyName}}</dd>
            </dl>
            <dl>
              <dt>责任系数</dt>
              <dd>{{data.dutyCoefficient}}</dd>
            </dl>
          </div>
        </div>
        <div class="taskArea detailArea" v-if="data.touchHistoryList.length">
          <dl>
            <dt>接触历史</dt>
          </dl>
          <div class="commuBox"  style="margin-top: -0.08rem">
            <div class="rateCont c" v-for="(item, index) in data.touchHistoryList">
              <dl class="rateConTime">
                <dt>{{item.touchDate | spliceDate}}</dt>
                <dd>{{item.touchDate | spliceTime}}</dd>
              </dl>
              <dl class="rateConTxt icon_rate" :class="{'icon_time': index===0}">
                <dt style="word-break: break-all;">{{item.touchResult}}<span v-if="item.touchDetail">--{{item.touchDetail}}</span>
                </dt>
                <dd>操作人：{{item.operator}}（<a :href="'tel:'+item.operatorPhone" otype="button"
                                             otitle="152-7441-1234">{{item.operatorPhone | phoneType}}</a>）</dd>
              </dl>
            </div>
            <!--<div v-else style="text-align: center;padding: 0.5rem 0; color:#999;">暂无接触记录</div>-->
          </div>
        </div>
      </div>
    </div>

    <!-- 第一次获取的数据为空时显示的页面 -->
    <div v-if="showDataNull" class="dataNullWrap">
      <div class="imgWrap"></div>
      <div class="dataNullText">{{errText}}</div>
    </div>

    <!-- 承保险别详情 -->
    <pts-alert v-model="showInsureMsg">
      <div class="reMaskClaim reMaskReport dn" style="display: block;">
        <a href="javascript:;" otype="button" otitle="关闭" class="closeClaimMask" @click="showInsureMsg = !showInsureMsg">关闭</a>
        <h3>承保险别详情</h3>
        <div class="reMaskCon">
          <table border="0" cellspacing="0" cellpadding="0" width="100%">
            <thead></thead>
            <tbody>
              <tr>
                <th width="62%">承保险别</th>
                <th width="38%">保额（元）</th>
              </tr>
              <tr v-for="(item,i) in data.planList" :key="i">
                <td><p>{{item.dutyName}}</p></td>
                <td>¥{{item.payLimit}}</td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </pts-alert>
  </div>
</template>

<script>
  import toast from '../../common/comComponent/toast'
  import axios from '../../common/js/axiosConfig'
  import API from '../../common/js/comConfig'
  import '../../common/filters/convertAmount'
  import ptsAlert from '../../common/comComponent/alertWindow'

  export default {
    name: "inRepairTaskInfo",
    data () {
      return {
        showInsureMsg: false,
        showDataNull: false,
        data: {},
        userNumList: '',
        isCaseImg:false, //判断有无案件照片
        imgArr:[], //存储图片ID
        errText: '查询失败, 退出重试',
      }
    },
    methods: {
      shareCase(){
        const _this = this;
        const u = navigator.userAgent;
        const isAndroid = u.indexOf('Android') > -1 || u.indexOf('Adr') > -1; //android终端
        const isiOS = !!u.match(/\(i[^;]+;( U;)? CPU.+Mac OS X/); //ios终端

        let shareData = {
          "title": "您有一个推修案件,请查看",
          "desc": "车牌: "+_this.data.carMark,
          "imgUrl": _this.data.icoUrl,
          "url": _this.data.shareUrl,
          "from": "平安好伙伴"
        };
        if(isiOS){
          let iframe = document.createElement("iframe");
          iframe.src = 'pagp://share|'+encodeURIComponent(window.JSON.stringify(shareData));
          iframe.style.display = "none";
          document.body.appendChild(iframe);
          iframe.parentNode.removeChild(iframe);
          iframe = null;
          window.eventAnalytics('内部推修任务','跳转案件详情分享')
        }else if(isAndroid){
          window.location.href = 'pagp://share|'+JSON.stringify(shareData);
          window.eventAnalytics('内部推修任务','跳转案件详情分享')
        }else{
          toast('暂不支持，请尝试在微信内打开！');
        }
      },

      getData (carMark, reportNo, taskId, taskStatus, vehicleFrameNo) {
        axios.post(API.getWebServiceUrls('pushRepairDetail'), {
          "carMark": carMark,
          "reportId": reportNo,
          "taskId": taskId,
          "taskStatus": taskStatus,
          "vehicleFrameNo": vehicleFrameNo
        }).then(res => {
          // console.log(res);
          let data = typeof res.data === 'string' ? JSON.parse(res.data) : res.data;
          switch (data.code) {
            case 0:
              this.data = data.data ? data.data : {};
              if (!data.data.caseStatus && !data.data.reportName) {
                this.showDataNull = true;
              };
              if(data.data.fileIdList.length > 0){
                this.isCaseImg = true;
                this.imgArr = data.data.fileIdList;
              };
              break;
            default:
              this.showDataNull = true;
              this.errText = '查询错误, 请重试';
              toast(data.msg);
              break
          }
        }).catch(err => {
          this.showDataNull = true;
          this.errText = '查询错误, 请重试';
          console.log(err);
        })
      },

      goBack () {
        if (window.history.length > 1) {
          window.history.go(-1);
        } else {
          Native.requestHybrid({
            tagname: 'backHome'
          })
        }
      },

      showInsureMsgEvent () {
        this.showInsureMsg = true;
        window.eventAnalytics('内部推修任务', '推修详情查看投保详情');
      },

      goShowBanner(){
        if(this.isCaseImg){
          const obj = this.$route.query;
          this.$router.push({path:'/inside/inRepairShowImage',
            query:{
              carMark:obj.carMark,
              reportNo:obj.reportId,
              vehicleFrameNo:obj.vehicleFrameNo,
              caseImgArr:this.imgArr
            }
          });
        }else{
          return false
        }
        window.eventAnalytics('内部推修任务', '进入案件照片');
      },
    },
    created () {
      let query = this.$route.query;
      this.getData(query.carMark, query.reportId, query.taskId, query.taskStatus, query.vehicleFrameNo);
    },
    components: {
      ptsAlert
    },
    filters: {
      fourSpace (v) {
        if (!v) {
          return v
        }
        let arr = v.split('');
        for (let i = 0; i < arr.length; i++) {
          if (i % 5 === 0) {
            arr.splice(i, 0, ' ')
          }
        }
        return arr.join('')
      },
      phoneType (v) {
        if (!v) {
          return v
        }
        let arr = (v + '').split('')
        if (arr.length > 3) {
          arr.splice(3, 0, '-')
        }
        if (arr.length > 7) {
          arr.splice(8, 0, '-')
        }
        return arr.join('')
      },
      spliceDate (v) {
        if (!v) return v;
        const index = v.indexOf('-')
        if (index < 0) {
          return v
        }
        return v.substr(index + 1, 5)
      },
      spliceTime (v) {
        if (!v) return v;
        const index = v.indexOf(' ')
        if (index < 0) {
          return v
        }
        return v.substr(index + 1, v.length)
      },
      vehicleLicenceCodeHidden (value) {
        if (!value) return value
        if (value.indexOf('*') > -1) {
          return '*_*'
        } else {
          return value
        }
      }
    }
  }
</script>

<style lang="less" scoped>
  @height:3.68rem;
  .reMaskCon{
    height: @height;
    overflow-y: scroll;
  }
  .nav_share_icon{
    display: inline-block;
    width: .45rem;
    height: .45rem;
    background: url("../../common/images/share_icon@3x.png") no-repeat;
    background-size: 100%;
    margin-top: -0.5rem;
    margin-right: .27rem;
  }
  .active{
    display: block;
    content: "";
    position: absolute !important;
    width: .69rem;
    height: .68rem;
    background: url("../../common/images/icon_haveImg@3x.png") no-repeat;
    background-size: .69rem .68rem;
    right: .31rem;
    top: 1rem;
  }
</style>
