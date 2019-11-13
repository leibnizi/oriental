<template>
  <div class="yuebao-more morepages">
      <!-- 头部 -->
      <div class="public-header">
        <div class="common-header hea-default-col">
            <div class="header-content">
                <div class="head-left" @click="goback()">
                    <i class="yo-icon icon-arr-left"></i>
                </div>
                <div class="head-title">转入记录</div>
                <div class="head-right">
                  <a class="font14" href="javascript:;" @click="showPopup()">状态筛选</a>
                </div>
            </div>
        </div>
      </div>

      <!-- 中心滑动区 -->
      <div class="loadmore-box" v-if="loaded && recordList.length !== 0">

        <mt-loadmore :top-method="loadTop" :bottom-method="loadBottom" :bottom-all-loaded="allLoaded" :auto-fill="false" ref="loadmore" @top-status-change="handleTopChange" @bottom-status-change="handleBottomChange">

            <!-- 列表 -->
            <dl class="public-list transfer-record">
              <dd v-for="(item, index) in recordList" :key="index" @click="showDetailData(item)" >
                <h4>
                  <span>{{ item.status | statusFilter }}</span>
                  <span class="time">{{ item.depositTimeStr }}</span>
                  <span v-if="item.tradeType === 201">余额转入</span>
                  <span v-else>人工存入</span>
                </h4>
                <div class="content-pt">
                  <h3>
                    <p>本金：¥{{ item.amount | toFixed }}</p>
                    <span>基准利率：{{ item.interestRate | moneyFilter}}%</span>
                  </h3>
                  <h3>
                    <p style="color: #F98527">收益：¥{{ item.interest | toFixed }}</p>
                    <span>额外利率：{{ item.additionInterestRate | moneyFilter}}%</span>
                  </h3>
                </div>
              </dd>
              <!-- 新增空白行 高度刚好等于底部切换按钮的高度 -->
              <!-- <dd class="last-list" style="height:60px; background-color:transparent">
              </dd> -->
            </dl>

          <div slot="top" class="mint-loadmore-top">
            <span v-show="topStatus !== 'loading'" :class="{ 'rotate': topStatus === 'drop' }">
                                        <i class="yo-icon icon-load-top"></i>
                                        <em>松开刷新....</em>
                                    </span>
            <mt-spinner color="#585252" :type="3" v-show="topStatus === 'loading'"></mt-spinner>
          </div>
          <div slot="bottom" class="mint-loadmore-bottom" style="display: block">
            <mt-spinner color="#585252" :type="3" v-show="bottomStatus === 'loading'"></mt-spinner>
          </div>
        </mt-loadmore>
      </div>
      <!--列表加载的loading-->
      <loadTop v-if="!loaded"></loadTop>

      <!--无数据时显示-->
      <no-data v-if="loaded && recordList.length === 0"></no-data>



    <!-- 确定按钮 -->
    <div class="public-footer common-footer reg-foot" v-if="!showDetail">
        <!-- 内容页页脚 -->
        <div class="but-footer yow-content succ-btn">
          <!-- btn-primary = 按钮正常状态， btn-disabled = 按钮禁用状态 -->
            <a href="javascript:void(0);" @click="toTransfer(1)" class="btn btn-sm btn-default">转出</a>
            <a href="javascript:void(0);" @click="toTransfer(2)" class="btn btn-sm btn-primary" :class="this.isOpenAccount === 0 ? 'closeState' : ''">转入</a>
        </div>
    </div>

    <!-- 类型pop筛选 -->
    <mt-popup v-model="popupVisible" position="bottom">
      <div class="public-pop plat-pop">
        <div class="public-pop-title">
          <a href="javascript:void(0);" @click="popupVisible = false">取消</a>
          <h3>类型筛选</h3>
          <a class="corr-col" href="javascript:void(0);" @click="popupConfig">确定</a>
        </div>
        <div class="type-selection clearfix">
          <span :class="statusList.indexOf(1) > -1 ? 'cur' : ''" @click="chooseType(1)">进行中</span>
          <span :class="statusList.indexOf(4) > -1 ? 'cur' : ''" @click="chooseType(4)">赎回中</span>
          <span :class="statusList.indexOf(3) > -1 ? 'cur' : ''" @click="chooseType(3)">已赎回</span>
          <span :class="statusList.indexOf(2) > -1 ? 'cur' : ''" @click="chooseType(2)">已终止</span>
          <span :class="statusList.indexOf(5) > -1 ? 'cur' : ''" @click="chooseType(5)">已撤销</span>
        </div>
      </div>
    </mt-popup>

    <!-- 详情页 -->
    <div v-if="showDetail" class="detail-page fixed-box">
      <!-- 头部 -->
      <div class="public-header">
        <div class="common-header hea-default-col">
            <div class="header-content">
                <div class="head-left" @click="showDetail = false">
                    <i class="yo-icon icon-arr-left" ></i>
                </div>
                <div class="head-title">详情</div>
                <div class="head-right"></div>
            </div>
        </div>
      </div>
      <!-- 列表 -->
      <div class="details-main main-box float-main">
        <div class="common-list">
          <dl class="enter-list detail-list">
            <dd>
              <h2>单号</h2>
              <span class="cont-txt color999">{{ detailData.depositBillNo }}</span>
            </dd>
            <dd>
              <h2>交易时间</h2>
              <span class="cont-txt color999">{{ detailData.depositTimeStr }}</span>
            </dd>
             <dd>
              <h2>交易类型</h2>
              <span v-if="detailData.tradeType === 201" class="cont-txt color3434">余额转入</span>
              <span v-else class="cont-txt color3434">人工存入</span>
            </dd>
            <dd>
              <h2>本金</h2>
              <span class="cont-txt color3434">￥{{ detailData.amount | toFixed }}</span>
            </dd>
            <dd>
              <h2>收益</h2>
              <span class="cont-txt colorF985">￥{{detailData.interest | toFixed}}</span>
            </dd>
            <dd>
              <h2>基本利率</h2>
              <span class="cont-txt color999">{{detailData.interestRate | moneyFilter}}%</span>
            </dd>
            <dd>
              <h2>额外利率</h2>
              <span class="cont-txt color999">{{detailData.additionInterestRate | moneyFilter}}%</span>
            </dd>
            <dd>
              <h2>状态</h2>
              <span class="cont-txt color3434">{{detailData.status | statusFilter}}</span>
            </dd>
            <dd>
              <h2>终止时间</h2>
              <span class="cont-txt">{{detailData.endTimeStr}}</span>
            </dd>
            <dd>
              <h2>赎回时间</h2>
              <span class="cont-txt">{{detailData.withdrawTimeStr}}</span>
            </dd>
            <dd class="item-cont">
              <h2>备注</h2>
              <span class="cont-txt color999">{{detailData.remark}}</span>
            </dd>
          </dl>

          <div class="magt10 text-center pay-btn" v-if="detailData.status === 1 || detailData.status === 2">
            <a class="btn btn-xl btn-primary" href="javascript:;" @click="withdraw">转出</a>
          </div>

        </div>
      </div>
    </div>
  </div>
</template>

<style>
.mint-loadmore-bottom .mint-spinner-fading-circle{
  margin-left: 45%;
}
</style>

<script>
import * as type from '@/base/store/yuebao/type'
import * as statusCode from '@/base/utils/status_const'
import * as payment from '@/base/store/payment/type'
import Vue from 'vue'
import Popup from 'mint-ui/lib/popup'
import Loadmore from 'mint-ui/lib/loadmore' // 按需加载mint-ui的模块
import Spinner from 'mint-ui/lib/spinner' // 按需加载mint-ui的模块
import noData from '@/base/components/statusPage/noData.vue'
import loadTop from '@/base/components/loadTop'
Vue.component(Popup.name, Popup)
Vue.component(Spinner.name, Spinner)
Vue.component(Loadmore.name, Loadmore)
export default {
  data () {
    return {
      showDetail: false,
      detailData: {},
      popupVisible: false,
      statusList: [],
      pageNo: 1,
      topStatus: '',
      bottomStatus: '',
      allLoaded: false,
      recordList: [],
      loaded: false,
      ischeckType: false // 当用户选择类型进行筛选的时候 pageNo 为1
    }
  },
  computed: {
    imgUrl () {
      return this.$store.state.home.commonImgUrl
    },
    isOpenAccount () {
      return this.$store.state.yuebao.isOpenAccount
    }
  },
  filters: {
    toFixed (num) {
      return num.toFixed(2)
    },
    statusFilter (status) {
      switch (status) {
        case 1:
          return '进行中'
        case 2:
          return '已终止'
        case 3:
          return '已赎回'
        case 4:
          return '赎回中'
        case 5:
          return '已撤销'
      }
    }
  },
  methods: {
    goback () {
      this.$router.push('/wap/yuebao/yuebaoHome')
    },
    toUrl (url) {
      this.$store.state.yuebao.currentFundsPool = {}
      this.$store.state.yuebao.yuebaoFundsPoolId = null
      this.$router.push(url)
    },
    transferIn (url) {
      this.$store.state.yuebao.currentFundsPool = {}
      this.$store.state.yuebao.yuebaoFundsPoolId = null
      if (this.isOpenAccount === 0) {
        return false
      }
      this.$router.push(url)
    },
    toTransfer (num) {
      // 转入的时候如果后台设置了关闭余额宝则不能进入转入页面
      this.$store.state.yuebao.currentFundsPool = {}
      this.$store.state.yuebao.yuebaoFundsPoolId = null
      // 记录跳转的分页
      this.$store.state.yuebao.toTransferPage = 1
      if (num === 2) {
        if (this.$store.state.yuebao.isOpenAccount === 0) {
          return false
        }
      }
      this.$store.dispatch(payment.INIT_DRAW_DATA_ACTION, {bankType: 0, 'loading': 'pass', 'isyuebao': '1'}).then(res => {
        if (res.status === statusCode.statusConst.SUCCESS) {
          if (num === 1) {
            window.router.push('/wap/yuebao/transferOut/toBalance')
          } else if (num === 2) {
            window.router.push('/wap/yuebao/transferIn')
          }
        }
      })
    },
    showDetailData (item) {
      console.log(item)
      this.detailData = item
      this.showDetail = true
    },
    withdraw () {
      this.$store.state.yuebao.currentFundsPool = this.detailData
      this.$store.state.yuebao.yuebaoFundsPoolId = this.detailData.id
      this.$store.dispatch(payment.INIT_DRAW_DATA_ACTION, {bankType: 0, 'loading': 'pass', 'isyuebao': '1'}).then(res => {
        if (res.status === statusCode.statusConst.SUCCESS) {
          // 记录跳转的分页
          window.store.state.yuebao.toTransferPage = 1
          window.router.push('/wap/yuebao/transferOut/toBalance')
        }
      })
    },
    showPopup () {
      this.popupVisible = true
    },
    popupConfig () {
      this.popupVisible = false
      this.loadTop()
    },
    chooseType (type) {
      if (this.statusList.indexOf(type) > -1) {
        this.statusList.splice(this.statusList.indexOf(type), 1)
      } else {
        this.statusList.push(type)
      }
      this.ischeckType = true
    },
    handleTopChange (status) {
      this.topStatus = status
    },
    handleBottomChange (status) {
      this.bottomStatus = status
    },
    loadTop () {
      // 加载更多数据
      if (this.ischeckType) {
        this.pageNo = 1
      }
      this.pageNo >= 2 ? this.pageNo -- : this.pageNo = 1
      this.recordList = []
      this.loadData(0)
      this.allLoaded = false // 下拉加载后重置滑动加载
      this.ischeckType = false
    },
    loadBottom () {
      // 加载更多数据 点击筛选以后从pageNo = 1 开始计算
      if (this.ischeckType) {
        this.pageNo = 1
      }
      this.pageNo++
      this.loadData(1)
      this.ischeckType = false
    },
    /**
     * @augments way 0:下拉刷新  1：上拉加载
     *
     */
    loadData (way) {
      // 用户点击筛选pageNo 为 1
      if (this.ischeckType) {
        this.pageNo = 1
      }
      this.loaded = false
      this.$store.dispatch(type.FUNDS_POOL_RECORD_ACTION, {'loading': 'pass', 'pageNo': this.pageNo, 'statusList': this.statusList}).then(res => {
        this.loaded = true
        if (res.status === statusCode.statusConst.SUCCESS) {
          if (res.data.list.length === 0) {
            this.allLoaded = true // 若数据已全部获取完毕
          }
          this.recordList = this.recordList.concat(res.data.list)
        }
        if (Number(way) === 0) {
          try {
            this.$refs.loadmore.onTopLoaded()
          } catch (e) {
          }
        } else if (Number(way) === 1) {
          try {
            this.$refs.loadmore.onBottomLoaded()
          } catch (e) {
          }
        }
      }, res => {
        this.loaded = true
      })
    }
  },
  mounted () {
    this.recordList = []
    this.loadData()
  },
  components: {
    noData,
    loadTop
  }
}
</script>

