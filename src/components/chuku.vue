<template>
  <div>
    <MyFilter url="http://doclever.cn:8090/mock/5c62e01a3dce46264b25bf54/getChukuSupplier"></MyFilter>
    <div class="date-wrap">
      <span class="show">{{this.date?this.date:initDate()}}</span>
      <div class="data-plugin-wrap">
        <input type="date" @input="getDate($event)">
        <i class="date-icon"></i>
      </div>
    </div>
    <div class="list-wrap" ref="wrapper">
      <div class="wrap">
        <ul class="content">
          <li v-for="(item,index) in list" :key="index">
            <div class="head">
              <div class="left">
                <span class="materiel-name">{{controlLength(item.materialname,6)}}</span>
                <!--<span class="materiel-id">({{controlLength(item.tabNumber,20)}})</span>-->
              </div>
              <div class="right">
                <span>-{{item.chukuNum}}</span>
                <!--<i class="normal-icon"></i>-->
              </div>
            </div>
            <div class="supplier">{{item.clientname}}</div>
            <div class="bottom">
              <div class="label-wrap">
                <span class="label">{{item.contractNumber || '暂无数据'}}T</span>
                <!--<span class="label">当前库存{{item.currentNum || '暂无数据'}}</span>-->
              </div>
              <div class="more">
                <router-link :to="{name: 'chukudetail',
                 params: {
                    materialname: item.materialname,
                    clientname:item.clientname,
                    data:item.data,
                    contractNumber:item.contractNumber,
                    chukuNum:item.chukuNum,
                    assistQty:item.assistQty,
                    taxPrice:item.taxPrice,
                    remark:item.remark,
                    model:item.model,
                 }
                }">更多</router-link>
              </div>
            </div>
          </li>
        </ul>
        <div class="loading-wrapper" v-show="showLoadingWrap">
          <p class="line"></p>
          <img src="../assets/loading-bottom.gif" alt="" v-show="showLoadingImg">
          <span v-show="!showLoadingImg">暂无数据</span>
          <p class="line"></p>
        </div>
      </div>
    </div>
  </div>
</template>

<script>

  import axios from 'axios'
  import BScroll from 'better-scroll'
  import MyFilter from './myfilter'

  export default {
    name: "chuku",
    components: {
      MyFilter,
    },
    data() {
      return {
        date: '',
        list: [],
        showLoadingWrap: false,
        showLoadingImg: false,
      }
    },
    methods: {
      controlLength(str, len) {
        console.log(str)
        if(str){
          return str.length > 6 ? str.slice(0, len) + '...' : str
        }
      },
      initDate() {
        let date = new Date();
        let seperator1 = "-";
        let year = date.getFullYear();
        let month = date.getMonth() + 1;
        let strDate = date.getDate();
        if (month >= 1 && month <= 9) {
          month = "0" + month;
        }
        if (strDate >= 0 && strDate <= 9) {
          strDate = "0" + strDate;
        }
        this.date = year + seperator1 + month + seperator1 + strDate;
      },
      getDate(e) {
        this.date = e.target.value
        this.loadData(1, 10)
      },
      loadData(page, pageAmount, materiel, organization, date, supplier) {
        this.showLoadingWrap = true;
        this.showLoadingImg = true
        axios.get('http://doclever.cn:8090/mock/5c62e01a3dce46264b25bf54/getChukuListMock', {
          params: {
            page: page,
            pageAmount: pageAmount,
            materiel: materiel,
            organization: organization,
            date: date,
            supplier: supplier,
          }
        })
          .then((response) => {
            console.log(response.data.data)
            if (response.data.data.length > 0) {
              this.showLoadingWrap = false;
              this.page++
              this.list = response.data.data.concat(this.list);
              console.log('list', this.list)
              this.$nextTick(() => {
                if (!this.scroll) {
                  this.scroll = new BScroll(this.$refs.wrapper, {
                    click: true,
                    pullUpLoad: {
                      threshold: -50 // 当上拉距离超过30px时触发 pullingUp 事件
                    }
                  })
                  this.scroll.on('pullingUp', () => {
                    this.loadData(this.page, 10)
                    this.scroll.finishPullUp()
                    // 事情做完，需要调用此方法告诉 better-scroll 数据已加载，否则上拉事件只会执行一次
                  })
                } else {
                  this.scroll.refresh()
                }
              })
            } else {
              this.showLoadingImg = false
            }

          })
          .catch((error) => {
            this.showLoadingImg = false
          })

      },
    },
    beforeRouteEnter(to, from, next) {
      next((vm) => {
        vm.list = []//请求数据前先清空当前数据，防止数据叠加
        vm.loadData(1, 10, vm.chosenMaterielList, vm.chosenOrganizationList, vm.date, vm.selectedResultId)
      })
    },
  }
</script>

<style scoped>
  .date-wrap {
    display: flex;
    justify-content: space-between;
    align-items: center;
    background-color: #F2F6F9;
    /*border-bottom: 1px solid #e9e9e9;*/
  }

  .date-wrap::after {
    content: "  ";
    position: absolute;
    left: 0;
    bottom: 0;
    width: 100%;
    height: 1px;
    background-color: #e9e9e9;
    /* 如果不用 background-color, 使用 border-top:1px solid #e0e0e0; */
  }

  @media screen and (-webkit-min-device-pixel-ratio: 2) {
    .date-wrap::after {
      transform: scaleY(.5);
    }
  }

  @media screen and (-webkit-min-device-pixel-ratio: 3) {
    .date-wrap::after {
      transform: scaleY(.33333);
    }
  }

  .date-wrap .show {
    padding: 0.08rem 0 0.08rem 0.12rem;
    font-size: 0.12rem;
    color: #030303;
  }

  .date-wrap .data-plugin-wrap {
    margin-right: 0.12rem;
    position: relative;
    font-size: 0.12rem;
    color: #030303;
  }

  .date-wrap .data-plugin-wrap input {
    position: absolute;
    top: 0;
    right: 0;
    opacity: 0;
    -webkit-tap-highlight-color: rgba(0, 0, 0, 0);
  }

  .date-wrap .data-plugin-wrap .date-icon {
    display: inline-block;
    width: 0.11rem;
    height: 0.11rem;
    background: url("../assets/date.png") no-repeat;
    background-size: contain;
  }

  .list-wrap {
    /*height: calc(100% - 0.8rem);*/
    height: calc(100% - 1.2rem);
    overflow: hidden
  }

  .list-wrap li {
    margin-left: 0.12rem;
    padding-top: 0.18rem;
    padding-bottom: 0.19rem;
    padding-right: 0.12rem;
    border-bottom: 1px solid #efefef;
    font-size: 0.1rem;
  }

  .list-wrap li::after {
    content: "  ";
    position: absolute;
    left: 0;
    bottom: 0;
    width: 100%;
    height: 1px;
    background-color: #efefef;
    /* 如果不用 background-color, 使用 border-top:1px solid #e0e0e0; */
  }

  @media screen and (-webkit-min-device-pixel-ratio: 2) {
    .list-wrap li::after {
      transform: scaleY(.5);
    }
  }

  @media screen and (-webkit-min-device-pixel-ratio: 3) {
    .list-wrap li::after {
      transform: scaleY(.33333);
    }
  }

  .list-wrap .content li .head {
    margin-bottom: 0.12rem;
    display: flex;
    justify-content: space-between;
    align-items: center;
  }

  .list-wrap .content li .head .left {
    display: flex;
    justify-content: space-between;
    align-items: center;
  }

  .list-wrap .content li .head .left .materiel-name {
    font-size: 0.14rem;
    color: #030303;
  }

  .list-wrap .content li .head .left .materiel-id {
    font-size: 0.12rem;
    color: #4b4b4b;
  }

  .list-wrap .content li .head .right {
    display: flex;
    justify-content: space-between;
    align-items: center;
  }

  .list-wrap .content li .head .right > span {
    font-size:0.14rem;
    color:#007aff;
  }

  .list-wrap .content li .head .right .normal-icon {
    display: inline-block;
    width: 0.14rem;
    height: 0.14rem;
    background: url("../assets/normal.png") no-repeat;
    background-size: contain;
  }

  .list-wrap .content li .supplier {
    margin-bottom: 0.09rem;
    font-size: 0.12rem;
    color: #4b4b4b;
  }

  .list-wrap .content li .bottom {
    display: flex;
    justify-content: space-between;
    align-items: center;
  }

  .list-wrap .content li .bottom .label-wrap span {
    margin-right: 0.14rem;
    padding: 0.03rem 0.14rem;
    display: inline-block;
    font-size: 0.11rem;
    color: #3897ff;
    background-color: #d4e9ff;
    border-radius: 0.06rem;
  }

  .list-wrap .content li .bottom .more a {
    font-size: 0.09rem;
    color: #bbbbbb;
  }

  .list-wrap .loading-wrapper {
    display: flex;
    justify-content: center;
    align-items: center;
    height: 0.8rem;
    background-color: #F2F6F9;
  }

  .list-wrap .loading-wrapper .line {
    height: 0px;
    width: 0.85rem;
    position: relative;
  }

  .list-wrap .loading-wrapper .line::after {
    content: "  ";
    position: absolute;
    left: 0;
    bottom: -0.02rem;
    width: 100%;
    height: 0px;
    background-color: #cacaca;
    border-bottom: 1px solid #cacaca;
    /* 如果不用 background-color, 使用 border-top:1px solid #e0e0e0; */
  }

  @media screen and (-webkit-min-device-pixel-ratio: 2) {
    .list-wrap .loading-wrapper .line::after {
      transform: scaleY(.5);
    }
  }

  @media screen and (-webkit-min-device-pixel-ratio: 3) {
    .list-wrap .loading-wrapper .line::after {
      transform: scaleY(.33333);
    }
  }

  .list-wrap .loading-wrapper img {
    margin: 0 0.08rem;
    width: 0.25rem;
    height: 0.25rem;
  }

  .list-wrap .loading-wrapper span {
    margin: 0 0.08rem;
    font-size: 0.09rem;
    color: #999999;
  }
</style>
