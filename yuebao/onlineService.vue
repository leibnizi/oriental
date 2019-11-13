<template>
  <div id="onlineService">
      <!-- 头部 -->
      <div class="public-header ">
        <div class="common-header">
            <div class="header-content">
                <div class="head-left" @click="goback()">
                    <i class="yo-icon icon-arr-left"></i>
                </div>
                <div class="head-title">我的客服</div>
                <div class="head-right">
                  <a href="javascript:void(0);" @click="openOnlineService()">
                    <i class="yo-icon icon-service"></i>
                    <span>客服</span>
                  </a>
                </div>
            </div>
        </div>
      </div>

      <!-- 不可滑动区 -->
      <div class="service-bj">
        <p>Hi<br>欢迎来到我的客服</p>
      </div>

      <!-- 滑动区 -->
      <div class="onlineServiceBox mainCenter">
        <h5 class="headline-txt" style="margin-left:.25rem;">猜你的问题</h5>
        <!-- 问题列表 -->
        <div class="common-list question-list">
          <dl class="enter-list">
            <dd @click="showDetail(1)" :class="cur === 1 ? 'cur' : ''">
              <div class="formIput">
                <span class="itemTxt">什么是余额宝？</span>
              </div>
              <div class="question-detail" v-if="cur === 1">
                余额宝是独立于娱乐城外的理财工具，单独核算单独审核，根据存入资金给予对应收益，旨在为客户带来更多的收益；
              </div>
            </dd>
            <dd @click="showDetail(2)" :class="cur === 2 ? 'cur' : ''">
              <div class="formIput">
                <span class="itemTxt">如何将余额转入余额宝？</span>
              </div>
              <div class="question-detail" v-if="this.cur === 2">
                手机登录应用，点击【我的】，选择余额宝/提款，点击转入/转入至余额宝，输入金额，确认即可;
              </div>
            </dd>
            <dd @click="showDetail(3)" :class="cur === 3 ? 'cur' : ''">
              <div class="formIput">
                <span class="itemTxt">余额宝收益如何计算？</span>
              </div>
              <div class="question-detail" v-if="this.cur === 3">
                余额宝按小时计息，转入余额宝后一天内按基准利率加额外利率计息，一天后按基准利率计息；
              </div>
            </dd>
            <dd @click="showDetail(4)" :class="cur === 4 ? 'cur' : ''">
              <div class="formIput">
                <span class="itemTxt">如何巧用余额宝？</span>
              </div>
              <div class="question-detail" v-if="this.cur === 4">
                提现至余额宝类似提款，可享受高额收益；余额宝转入至钱包余额类似充值；快快让你的资金动起来；
              </div>
            </dd>
            <dd @click="showDetail(5)" :class="cur === 5 ? 'cur' : ''">
              <div class="formIput">
                <span class="itemTxt">余额宝申明</span>
              </div>
              <div class="question-detail" v-if="this.cur === 5">
                余额宝旨在为用户创造更多收益，本公司对余额宝功能拥有最终解释权。
              </div>
            </dd>
          </dl>
        </div>

        <!--
        <div class="question-btn">
          <a :href="onLineService === undefined ? 'javascript:void(0)' : onLineService.junctionSim">
            在线服务
          </a>
        </div>
        -->

      </div>
  </div>
</template>

<script>
export default {
  data () {
    return {
      cur: ''
    }
  },
  methods: {
    openOnlineService () {
      // 判断是否是原生APP内嵌H5代码 客服协议
      window.indexMain.callService()
    },
    goback () {
      this.$router.go(-1)
    },
    showDetail (index) {
      if (this.cur === index) {
        this.cur = ''
      } else {
        this.cur = index
      }
    }
  },
  computed: {
    onLineService () {
      return this.$store.state.home.onlineservice
    }
  }
}
</script>

