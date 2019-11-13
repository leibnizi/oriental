<template>
    <div class="pane">
      <!-- 头部 -->
      <div class="public-header ">
        <div class="common-header hea-default-col">
            <div class="header-content">
                <div class="head-left" @click="goback()">
                    <i class="yo-icon icon-arr-left"></i>
                </div>
                <div class="head-title">转出</div>
                <!-- <div class="head-right">
                  <a :href="onLineService === undefined ? 'javascript:void(0)' : onLineService.junctionSim">
                    <i class="yo-icon icon-service"></i>
                  </a>
                </div> -->
            </div>
        </div>
      </div>

      <!-- 不可滑动区-主导航 -->
      <div class="fixed-menu fixed-box">
        <div class="common-tab list-tab2 menu-tab ">
          <ul class="clearfix">
            <li :class="{cur: this.nowType === 0}" @click="changeTab(0)">
              <a href="javascript:void(0);">转出至账户余额</a>
            </li>
            <li :class="{cur: this.nowType === 1}" @click="changeTab(1)">
              <a href="javascript:void(0);">转出至银行卡</a>
            </li>
          </ul>
        </div>
      </div>

      <!-- 滑动区 -->
      <div class="message-main main-box">
          <toBalance v-if="nowType === 0"></toBalance>
          <toBank v-if="nowType ===1"></toBank>
      </div>

  </div>
</template>

<script>
import toBalance from './toBalance'
import toBank from './toBank'
export default {
  data () {
    return {
      nowType: 0 // 当前显示的页面，0默认为余额宝，1为银行卡
    }
  },
  watch: {
    $route (to, from) {
      if (to.params && (to.params === 'toBalance' || to.params.id === 'toBalance')) {
        this.nowType = 0
      }
      if (to.params && (to.params === 'toBank' || to.params.id === 'toBank')) {
        this.nowType = 1
      }
    }
  },
  computed: {
    toTransferPage () {
      return this.$store.state.yuebao.toTransferPage
    },
    onLineService () {
      return this.$store.state.home.onlineservice
    },
    /** 后端传入值为0 和 1 */
    type () {
      return this.$store.state.home.type
    },
    userInfo () {
      return this.$store.state.home.userInfo
    }
  },
  components: {
    toBalance,
    toBank
  },
  methods: {
    goback () {
      if (this.toTransferPage === 0) {
        this.$router.push('/wap/yuebao/yuebaoHome')
      } else {
        this.$router.push('/wap/yuebao/yuebaoList')
      }
    },
    GoHome () {
      this.$router.push('/wap/index')
    },
    changeTab (nowType) {
      this.nowType = nowType
      this.$store.state.yuebao.currentFundsPool = {}
      this.$store.state.yuebao.yuebaoFundsPoolId = null
      if (nowType === 1) {
        window.router.push('/wap/yuebao/transferOut/toBank')
      } else {
        window.router.push('/wap/yuebao/transferOut/toBalance')
       // this.$authJumpUrl('wap/withdrawal', {needLogin: true, forbidTryUser: true, text: '提现'})
      }
    }
  },
  mounted () {
    if (this.$route.params.id === 'toBalance') {
      this.nowType = 0
    }
    if (this.$route.params.id === 'toBank') {
      this.nowType = 1
    }

    this.$store.state.teamRecord.openRecordBox = 0
    if (Number(this.type) === 1) {
      window.layer.msgWarn('该功能只对正式账号进行开放，请您注册正式账号！')
      this.$router.push('/wap/index')
    }
  }
}
</script>

