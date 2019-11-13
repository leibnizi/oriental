<template>
  <div class="in-come-expenses message-more morepages">
      <!-- 头部 -->
      <div class="public-header ">
        <div class="common-header hea-default-col">
            <div class="header-content">
                <div class="head-left" @click="goback()">
                    <i class="yo-icon icon-arr-left"></i>
                </div>
                <div class="head-title">收支明细</div>
                <div class="head-right"></div>
            </div>
        </div>
      </div>

      <!-- 不可滑动区 -->
      <div class="title-fixed fixed-box in-come-time">
        <div class=" clearfix">
          <span>交易时间</span>
          <div class="time-data-wrap">
            <!-- 时间插件 -->
            <time-date :start='starTime' :end='endTime'
                      :getSelectTime='getSelectTime' ref="timeRef"
                      :disabledTomorrow='minStartDate' :enabledToday='maxEndDate'></time-date>
          </div>
          <span @click="showPopup()" class="typeTxt">类型</span>
        </div>
      </div>

      <!-- 中心滑动区 -->
      <div class="loadmore-box " v-if="loaded">

      <mt-loadmore :top-method="loadTop"
                  :bottom-method="loadBottom"
                  :bottom-all-loaded="allLoaded"
                  :auto-fill="false" ref="loadmore"
                  @top-status-change="handleTopChange"
                  @bottom-status-change="handleBottomChange">

          <div class="income-list-box">
            <!-- 数据列表体 -->
            <dl class="public-list">
              <dd v-for="(item, index) in flowRecordList" :key="index">
                <h4>
                  <span></span>
                  <span>{{item.time}}</span>
                </h4>
                <div>
                  <h3><p>交易金额：¥{{item.tradeMoney.toFixed(2)}}</p><span>{{item.tradeTypeName}}</span></h3>
                  <span>单号：{{item.withdrawBillNo}}</span>
                </div>
              </dd>
            </dl>

            <!--无数据时显示 -->
            <no-data v-if="flowRecordList.length === 0 || isNoData"/>

          </div>

          <!-- 上拉加载，下拉刷新 -->
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
      <loadTop v-if="!loaded"></loadTop>


    <!-- 类型筛选 -->
    <mt-popup v-model="popupVisible" position="bottom">
      <div class="public-pop yuebao-pop-up">
        <div class="public-pop-title">
          <a href="javascript:void(0);" @click="popupVisible = false">取消</a>
          <h3>类型筛选</h3>
          <a class="corr-col" href="javascript:void(0);" @click="popupConfig">确定</a>
        </div>
        <dl class="platform-list payment-list type-selection" v-if="popupVisible">
          <dd v-for="(type, index) in types" @click="changeType(type)" :key="index" class="type-name">
            {{type.name}}
            <p v-show="type.status"><i class="yo-icon icon-correct corr-col"></i></p>
          </dd>
        </dl>
      </div>
    </mt-popup>
  </div>
</template>

<script>
import Vue from 'vue'
import moment from 'moment'
import * as type from '@/base/store/yuebao/type'
import * as cookieUtil from '@/base/utils/cookieUtil'
import Spinner from 'mint-ui/lib/spinner'
import Loadmore from 'mint-ui/lib/loadmore'
import Popup from 'mint-ui/lib/popup'
import timeDate from '@/web/components/memberCenter/timeDate'
import noData from '@/base/components/statusPage/noData.vue'
import loadTop from '@/base/components/loadTop'
Vue.component(Popup.name, Popup)
Vue.component(Spinner.name, Spinner)
Vue.component(Loadmore.name, Loadmore)
export default {
  data () {
    return {
      starTime: '', // 当前选择的开始时间
      endTime: '', // 当前选择的结束时间
      minStartDate: '', // 最早可选日期为上月第一天
      maxEndDate: '', // 最后可选日期为今天
      popupVisible: false,
      loaded: false,
      currenPlatformUtc: -4, // 当前选择的彩种使用时间的时区 用来计算时间偏移量和控制提示信息的显示和隐藏
      flowRecordList: [],
      types: [{
        value: '203',
        name: '转出至余额',
        status: false
      },
      {
        value: '204',
        name: '收益转出至余额',
        status: false
      },
      {
        value: '201',
        name: '余额转入',
        status: false
      },
      {
        value: '207',
        name: '实际转出至银行卡',
        status: false
      },
      {
        value: '208',
        name: '实际收益转出至银行卡',
        status: false
      },
      {
        value: '211',
        name: '余额宝人工存入',
        status: false
      },
      {
        value: '212',
        name: '余额宝活动',
        status: false
      },
      {
        value: '213',
        name: '余额宝存款优惠',
        status: false
      },
      {
        value: '214',
        name: '人工存入误存',
        status: false
      },
      {
        value: '215',
        name: '手动申请出款',
        status: false
      },
      {
        value: '216',
        name: '收益提出-人工误存',
        status: false
      },
      {
        value: '217',
        name: '收益提出-手动出款',
        status: false
      }
      ],
      pageNo: 1, // 分页
      pageSize: 20,
      allLoaded: false, // 加载 false可以加载 true不可以拖动加载
      topStatus: '', // 上滑状态
      bottomStatus: '', // 下滑状态
      operateType: '', // 选中的类型
      exdata: {
        txt: '暂无内容'
      },
      isNoData: false
    }
  },
  components: {
    timeDate,
    noData,
    loadTop
  },
  methods: {
    goback () {
      this.$router.go(-1)
    },
    // 点击确定的时候，设置开始，结束时间
    getSelectTime (start, end) {
      this.starTime = start
      this.endTime = end
      this.flowRecordList = []
      this.yuebaoFlowRecord(0)
    },
    showPopup () {
      this.popupVisible = true
    },
    popupConfig () {
      this.operateType = ''
      this.popupVisible = false
      this.flowRecordList = []
      this.pageNo = 1
      this.yuebaoFlowRecord(0)
    },
      /**
     *  按平台 UTC 计算当前各平台的时间
     */
    getAreTime (timeArea) {
      let d = new Date()
      let localTime = d.getTime() // 获取1970年1月1日后到此时时间之间的毫秒数
      let localOffset = d.getTimezoneOffset() * 60000 // 获取当地时间与格林尼治时间偏移值，转换成毫秒
      let utc = localTime + localOffset // 将当前时间与时区偏移量相加，得到国际标准时间， 毫秒
      let calctime = utc + (3600000 * timeArea) // timeArea为具体时区，美东标准时间为 -5，夏令时间为 -4；格林时间为 0；北京时间为 8
      return {
        day: new Date(calctime).getDay(),
        date: moment(new Date(calctime)).format('YYYY-MM-DD'),
        time: new Date(calctime)
      } // 日期格式转换
    },
    changeType (type) {
      this.$set(type, 'status', !type.status)
    },
    /**
     * 收支明细接口请求
     */
    yuebaoFlowRecord (status) {
      for (let i in this.types) {
        if (this.types[i].status) {
          if (this.operateType === '') {
            this.operateType = this.types[i].value
          } else {
            this.operateType += ',' + this.types[i].value
          }
        }
      }
      let params = {
        pageNo: this.pageNo,
        pageSize: this.pageSize,
        startDate: this.starTime,
        endDate: this.endTime,
        operateType: this.operateType,
        loading: 'pass' // 去掉调用接口的大loading
      }
      this.loaded = false
      this.$store.dispatch(type.QUERY_FLOW_RECORD_ACTION, params).then(res => {
        this.loaded = true
        if (res.status === '1') {
          if (res.data.length === 0) {
            this.allLoaded = true // 若数据已全部获取完毕
          } else {
            this.allLoaded = false
          }
          if (Number(status) === 0) { // 刷新取前20条
            this.flowRecordList = res.data
            try {
              this.$refs.loadmore.onTopLoaded()
            } catch (e) {
            }
          } else if (Number(status) === 1) { // 加载将所有页数据拼起来
            this.flowRecordList = this.flowRecordList.concat(res.data)
            this.$refs.loadmore.onBottomLoaded()
          }
          // 数据处理
          for (let i in this.flowRecordList) {
            this.$set(this.flowRecordList[i], 'time', cookieUtil.getFormatDate(this.flowRecordList[i].createTime))
          }

          // 没有数据
          if (this.flowRecordList.length === 0) {
            this.isNoData = true
          } else {
            this.isNoData = false
          }

          // // 计算是否有分页
          // let total = res.total
          // let nextlength = parseInt((total + this.pageSize - 1) / this.pageSize)
          // if (nextlength <= this.pageNo) {
          //   this.isload = false
          //   this.allLoaded = true
          // } else {
          //   this.isload = true
          //   this.allLoaded = false
          // }
        }
      }).catch(res => {
        this.loaded = true
        console.log(res, 'res')
      })
    },
    loadTop () {
      // 加载更多数据
      this.pageNo = 1
      this.flowRecordList = []
      this.yuebaoFlowRecord(0)
      this.allLoaded = false // 下拉加载后重置滑动加载
    },
    loadBottom () {
      // 加载更多数据
      this.pageNo++
      this.yuebaoFlowRecord(1)
      this.$refs.loadmore.onBottomLoaded()
    },
     // 上滑状态
    handleTopChange (status) {
      this.topStatus = status
    },
    // 下滑状态
    handleBottomChange (status) {
      this.bottomStatus = status
    }
  },
  computed: {
    imgUrl () {
      return this.$store.state.home.commonImgUrl
    }
  },
  mounted () {
    // 初始化时间
    this.starTime = this.getAreTime(this.currenPlatformUtc).date
    this.endTime = this.getAreTime(this.currenPlatformUtc).date
    this.$refs.timeRef.setNewTime(this.starTime, this.endTime)
    this.flowRecordList = []
    // 初始化数据体
    this.yuebaoFlowRecord(0)
  }
}
</script>

