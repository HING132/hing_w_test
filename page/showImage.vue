<template>
  <transition name="fade">
    <div class="banner_box" @click="goBack">
      <div class="header">
        <p>{{currentPage}}/{{totalPage}}</p>
      </div>
      <div class="swiper-wrap" ref="banner">
        <ul class="swiper-box c" ref="bannerList">
          <li class="swiper-item" v-for="(item,index) in imgSrcArr" :key="index">
            <img :src="str+item" alt="" style="width: 7.5rem;">
          </li>
        </ul>
      </div>
    </div>
  </transition>
</template>

<script>
    import API from '../../../common/js/comConfig'
    import Axios from '../../../common/js/axiosConfig'
    import toast from '../../../common/comComponent/toast'
    import Banner from '../../../common/js/ptsSwiper'

    export default {
      name: "inRepairShowImage",
      data(){
        return {
          currentPage:1,
          totalPage:1,
          str:'data:image/jpeg;base64,', //base64编码解析图片路径
          imgIdArr:[], //用来存储图片ID
          imgSrcArr:[], //用来存储图片src地址
          indexArr:[1], //存储每张图片的页码
        }
      },
      methods:{
        goBack(){
          if(window.history.length >= 1){
            window.history.go(-1);
          }
        },
        getData(i){
          const _this = this;
          Axios.post(API.getWebServiceUrls('pushRepairCaseImage')+'?fileId='+_this.imgIdArr[i])
            .then(res =>  {
              let data = typeof res.data === 'string' ? JSON.parse(res.data) : res.data;
              if(data.code === 0 || date.code === '0'){
                _this.imgSrcArr.push(data.data);
              }else{
                toast(data.msg)
              }
            }).catch(err => {
              console.log(err)
          })
        }
      },
      created(){
        let arr = this.$route.query.caseImgArr;
        this.imgIdArr = arr;
        this.totalPage = arr.length;
        this.getData(0); //获取第一张图片
        if (this.totalPage >= 2) this.getData(1); //如果有第二张的话也一起获取
      },
      mounted(){
        const _this = this;
        new Banner({
          box: _this.$refs.banner,
          ul: _this.$refs.bannerList,
          liWidth: Boolean, //是否需要插件自己计算ul 下的子元素的宽度 默认true
          callback: function(index){ // 返回当前是第几张图片;
            _this.currentPage = index+1;
          }
        })
      },
      watch:{
        currentPage (newVal,oldVal) {
          if (this.indexArr.indexOf(this.currentPage) > -1) return //保存每张图片的页码,如果已经存在,则return出去
          this.indexArr.push(this.currentPage);
          if (this.currentPage+1 <= this.totalPage) {
            this.getData(this.currentPage)
          }
        }
      }

    }
</script>

<style scoped lang="less">
  .fade-enter-active,
  .fade-leave-active {
    transition: all .3s linear;
    transform: scale(1)
  }

  .fade-enter,
  .fade-leave-active {
    transition: all .3s linear;
    transform: scale(0)
  }

  .banner_box{
    background: black;
    height: 100%;
    & .header{
      position: fixed;
      top: .44rem;
      left: 0;
      width: 100%;
      height: .5rem;
      background: black;
      font-family: PingFangSC-Regular;
      font-size: .28rem;
      color: #FFFFFF;
      line-height: .5rem;
      text-align: center;
      letter-spacing: 0.04rem;
    }
  }
  .swiper-wrap{
    top: 32%;
    position: relative;
    width: 100%;
    height: 5.64rem;
    overflow: hidden;
    & .swiper-box{
      position:absolute;
      width: 100000px !important;
      .swiper-item{
        float: left;
      }
    }
  }
</style>
