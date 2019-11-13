<template>
  <div id="yuebaoHome" class="pane">
      <!-- 头部 -->
      <div class="public-header">
        <div class="common-header">
            <div class="header-content">
                <div class="head-left" @click="goback()">
                    <i class="yo-icon icon-arr-left"></i>
                </div>
                <div class="head-title">余额宝</div>
                <div class="head-right"></div>
            </div>
        </div>
      </div>

      <!-- 不可滑动区 -->
      <div class="yuebao-bj"></div>

      <!-- 可滑动区域 -->
      <div class="common-main main-box">
          <!-- 收益总览 -->
          <div class="yuebao-info">
            <div class="yuebao-mony">
              <!--文字与icon不要换行，有些浏览器显示的空隙会很宽-->
              <p class="yuebao-position-adj">总余额（¥）<i class="yo-icon icon-pwd-show-k" v-if="showPws === '0'" @click="showPwsFun(1)"></i>
                <i class="yo-icon icon-pwd-hide" v-else @click="showPwsFun(0)"></i>
              </p>
              <h3 v-show="showPws === '0' && yuebaoInited">
                {{(yuebaoHome.totalBalance + yuebaoHome.nowInterest) | moneyFilter}}
              </h3>
              <h3 v-show="showPws === '0' && !yuebaoInited" class="yue-no-inited">— —</h3>
              <h3 v-show="showPws === '1'">****</h3>
            </div>
            <div class="yue-earnings">
              <div class="yuebao-mony">
                <p>总本金（¥）</p>
                <h3 v-show="showPws === '0' && yuebaoInited">{{yuebaoHome.totalBalance | moneyFilter}}</h3>
                <h3 v-show="showPws === '0' && !yuebaoInited" class="yue-no-inited">— —</h3>
                <h3 v-show="showPws === '1'">****</h3>
              </div>
              <div class="yuebao-mony">
                <p>当前收益（¥）</p>
                <h3 v-show="showPws === '0' && yuebaoInited">{{yuebaoHome.nowInterest | moneyFilter}}</h3>
                <h3 v-show="showPws === '0' && !yuebaoInited" class="yue-no-inited">— —</h3>
                <h3 v-show="showPws === '1'">****</h3>
              </div>
              <div class="yuebao-mony">
                <p>累计收益（¥）</p>
                <h3 v-show="showPws === '0' && yuebaoInited">{{(yuebaoHome.nowInterest + yuebaoHome.totalInterest) | moneyFilter}}</h3>
                <h3 v-show="showPws === '0' && !yuebaoInited" class="yue-no-inited">— —</h3>
                <h3 v-show="showPws === '1'">****</h3>
              </div>
            </div>
          </div>

          <!-- 利率 -->
          <div class="yue-interest-rate">
            <p>当前日利率：</p>
            <p v-if="yuebaoHome.yuebaoStationSetting">
              <span v-if="yuebaoInited">
                基准利率：{{yuebaoHome.yuebaoStationSetting.interestRate | moneyFilter}}%
              </span>
              <span v-if="!yuebaoInited">基准利率：— —</span>
              <span v-if="yuebaoInited">
                额外利率：{{yuebaoHome.yuebaoStationSetting.additionInterestRate | moneyFilter}}%
              </span>
              <span v-if="!yuebaoInited">额外利率：— —</span>
              </p>
          </div>

          <!-- 服务 -->
          <div class="yue-serve">
            <h5 class="headline-txt">我的服务</h5>
            <dl class="clearfix">
              <dd @click="toUrl('/wap/yuebao/yuebaoList')">
                <img :src="commonImgUrl + '/yuebao/zhuanru.svg'"/>
                <p>转入记录</p>
              </dd>
              <dd @click="toUrl('/wap/yuebao/incomeExpenses')">
                <img :src="commonImgUrl + '/yuebao/shouzhimingxi.svg'"/>
                <p>收支明细</p>
              </dd>
              <dd @click="toUrl('/wap/yuebao/onlineService')">
                <img :src="commonImgUrl + '/yuebao/yuebaoshuoming.svg'"/>
                <p>余额宝说明</p>
              </dd>
            </dl>
          </div>

      </div>

      <!-- 确定按钮 -->
      <div class="public-footer common-footer reg-foot">
          <!-- 内容页页脚 -->
          <div class="but-footer yow-content succ-btn">
            <!-- btn-primary = 按钮正常状态， btn-disabled = 按钮禁用状态 -->
              <a href="javascript:void(0);" @click="toTransfer(1)" class="btn btn-sm btn-default">转出</a>
              <a href="javascript:void(0);" @click="!(isOpenAccount ===0 || !yuebaoInited) && toTransfer(2)" class="btn btn-sm btn-primary"
               :class="(isOpenAccount ===0 || !yuebaoInited) ? 'closeState' : ''">转入</a>
          </div>
      </div>

  </div>
</template>
<script>
import * as type from '@/base/store/yuebao/type'
import * as payment from '@/base/store/payment/type'
import * as statusCode from '@/base/utils/status_const'
export default {
  data () {
    return {
      showPws: localStorage.getItem('showPws'), // 显示隐藏金额，0显示，1隐藏
      yuebaoInited: false // 余额宝接口是否已初始化
    }
  },
  computed: {
    commonImgUrl () { // 公共图片路径
      return this.$store.state.home.commonImgUrl
    },
    yuebaoHome () {
      return this.$store.state.yuebao.yuebaoHome
    },
    toPrePage () {
      return this.$store.state.yuebao.toPrePage
    },
    /** 后端传入值为0 和 1 */
    type () {
      return this.$store.state.home.type
    },
    isOpenAccount () {
      return this.$store.state.yuebao.isOpenAccount
    }
  },
  mounted () {
    // 查询首页数据
    this.$store.dispatch(type.INIT_HOME_ACTION, {'loading': 'pass'}).then((res) => {
      if (res.status === statusCode.statusConst.SUCCESS) {
        this.yuebaoInited = true
      }
    })
    // 默认显示金额
    if (!localStorage.getItem('showPws')) {
      localStorage.setItem('showPws', 0)
      this.showPws = localStorage.getItem('showPws') + ''
    }
    // 新增判断后台是否开启余额宝账户的设定
    this.$store.dispatch(type.YUEBAO_ACCOUNT_SETTING_ACTION, {'loading': 'pass'})
  },
  methods: {
    goback () {
      // 判断是否是原生APP内嵌H5代码 返回协议
      if (window.myNative && window.myNative.isApp()) {
        window.AppInvokeModel.popToUp()
        return true
      }
      // if (this.toPrePage === 0) {
      //   this.$router.push('/wap/index')
      // } else {
      this.$router.push('/wap/basicsInfo')
      // }
    },
    toUrl (url) {
      this.$router.push(url)
    },
    showPwsFun (type) {
      localStorage.setItem('showPws', type)
      this.showPws = type + ''
    },
    toTransfer (num) {
      this.$store.state.yuebao.currentFundsPool = {}
      this.$store.state.yuebao.yuebaoFundsPoolId = null
      // 记录跳转的分页，从首页跳转过去
      this.$store.state.yuebao.toTransferPage = 0
      // 转入的时候如果后台设置了关闭余额宝则不能进入转入页面
      if (num === 1) {
        window._dinWarmUp && window._dinWarmUp('send', 'event', 'Link', '余额宝_转出', '余额宝_转出')
        sessionStorage.setItem('switchBank', '/wap/yuebao/transferOut/toBalance')
      }
      if (num === 2) {
        window._dinWarmUp && window._dinWarmUp('send', 'event', 'Link', '余额宝_转入', '余额宝_转入')
        sessionStorage.setItem('switchBank', '/wap/yuebao/transferIn')
        if (this.isOpenAccount === 0) {
          return false
        }
      }
      this.$store.dispatch(payment.INIT_DRAW_DATA_ACTION, {bankType: 0, 'loading': 'pass', 'isyuebao': '1'}).then(res => {
          // 如果没有登录需要
        if (res.status === statusCode.statusConst.SUCCESS) {
          if (num === 1) {
            window.router.push('/wap/yuebao/transferOut/toBalance')
          } else if (num === 2) {
            window.router.push('/wap/yuebao/transferIn')
          }
        }
      })
    }
  }
}
</script>
