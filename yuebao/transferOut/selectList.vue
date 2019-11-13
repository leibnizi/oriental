<template>
  <div>
      <div class="common-list">
        <!-- 选择转出金额 -->
        <dl class="enter-list content-right" @click="clickList()">
          <dd>
            <h2>转出金额(￥)</h2>
            <p class="font14">
              <span v-if="finallyResult">{{finallyResult | moneyFilter}}</span>
              <span v-if="!finallyResult">请选择转出金额</span>
              <i class="yo-icon icon-arr-right"></i>
            </p>
          </dd>
        </dl>
      </div>

      <!-- 弹框 -->
      <mt-popup v-model="openSelect" position="bottom" attr-div="depositList">
        <div class="public-pop plat-pop">
          <div class="public-pop-title">
            <!-- <a href="javascript:void(0);" @click="openSelect = false" >取消</a> -->
            <h3>选择转出金额</h3>
            <!-- <a class="corr-col" href="javascript:void(0);" @click="clickList(1)">完成</a> -->
          </div>
          <dl class="platform-list slectMony">
            <!-- 总本金的方法和全选的方法 完全可以一致 ....-->
            <dd @click="chooseAll()">
              <span>总本金:{{totalBalance|moneyFilter}}</span>
              <span>当前收益:{{nowInterest|moneyFilter}}</span>
              <i class="yo-icon corr-col" :class="checkAll === true ? 'icon-correct':''"></i>
            </dd>
            <dd v-for="(fundsPool,index) in fundsPoolList" :key="index" @click="setFundsPool(fundsPool)">
              <span>本金:{{fundsPool.amount|moneyFilter}}</span>
              <span>收益:{{fundsPool.interest|moneyFilter}}</span>
              <span> {{fundsPool.depositTime | messageTime | contentFilterTime(5,20)}}</span>
              <i class="yo-icon corr-col icon-radio"
                  :class="(nowFundsPool.id === fundsPool.id )? 'icon-correct':''"></i>
            </dd>
          </dl>
        </div>
      </mt-popup>
    <!-- 总本金的方法和全选的方法 完全可以一致 ....-->
      <!-- <div class="wrapper2" pageId='live'>
        <div v-show="openStroe" attr-div="depositList" class="TipsWindown">
          <div class="shadeIn">
            <div class="selBankBox">
              <h1>选择转出金额
                <a href="javascript:;" @click="clickList(1)">完成</a>
              </h1>
              <dl class="fundsPool slectMony">

                <dt @click="chooseAll()">
                  <span style="width: calc((100% - 20px) / 2);">总本金:{{totalBalance|moneyFilter}}</span>
                  <span style="width: calc((100% - 20px) / 2);">当前收益:{{nowInterest|moneyFilter}}</span>
                  <i class="yo-icon icon-radio"
                      :class="checkAll === true ? 'icon-radio-yes':'icon-radio-no'"></i>
                </dt>
                <dl class="fundsPool">
                  <dt v-for="(fundsPool,index) in fundsPoolList" :key="index" @click="setFundsPool(fundsPool)">
                    <span>本金:{{fundsPool.amount|moneyFilter}}</span>
                    <span>收益:{{fundsPool.interest|moneyFilter}}</span>
                    <span style="width: calc(32%);">
                      {{fundsPool.depositTime | messageTime | contentFilterTime(5,20)}}
                      <i class="yo-icon icon-radio" style="font-size:20px;"
                        :class="(nowFundsPool.id === fundsPool.id )? 'icon-radio-yes':'icon-radio-no'"></i>
                    </span>
                  </dt>
                </dl>
              </dl>
            </div>
          </div>
        </div>
      </div> -->

  </div>
</template>

<script>
  import Vue from 'vue'
  import Popup from 'mint-ui/lib/popup'
  Vue.component(Popup.name, Popup)
  export default {
    data () {
      return {
        openSelect: false, // 打开选择转出金额弹框
        nowFundsPool: {}, // 当前页面选中的参数,
        checkAll: false,
        finallyResult: 0
      }
    },
    methods: {
      /**
       * 关闭存款方式下拉框
       */
      clickList () {
        this.openSelect = !this.openSelect
        // 确认时在做数据保存
        window.indexMain.appHiddenOverflow(true)
        window.$('input, textarea').blur()// 控制软键盘消失
      },
      setFundsPool (fundsPool) {
        this.openSelect = false
        this.checkAll = false
        // 赋值全局当前选中的参数对象
        this.nowFundsPool = JSON.parse(JSON.stringify(fundsPool))
        this.finallyResult = this.nowFundsPool.amount + this.nowFundsPool.interest
        this.$store.state.yuebao.yuebaoFundsPoolId = this.nowFundsPool.id
        this.$emit('getMoney', this.finallyResult)
        // 发送优惠金额
        this.$emit('toAmount', this.finallyResult)
      },
      // 点击全选
      chooseAll () {
        this.openSelect = false
        this.checkAll = true
        this.finallyResult = this.totalBalance + this.nowInterest
            // 全选则赋值yuebaoFundsPoolId为0
        this.$store.state.yuebao.yuebaoFundsPoolId = 0
        this.nowFundsPool.id = 0
        this.$emit('getMoney', this.finallyResult)
        // 发送优惠金额
        this.$emit('toAmount', this.finallyResult)
      }
    },
    computed: {
      type () {
        return this.$store.state.home.type
      },
      fundsPoolList () {
        return this.$store.state.yuebao.fundsPoolList
      },
      currentFundsPool () {
        return this.$store.state.yuebao.currentFundsPool
      },
      totalBalance () {
        return this.$store.state.yuebao.totalBalance
      },
      totalInterest () {
        return this.$store.state.yuebao.totalInterest
      },
      nowInterest () {
        return this.$store.state.yuebao.nowInterest
      },
      chooseBalance () {
        return this.$store.state.yuebao.chooseBalance
      }
    },
    mounted () {
      // 如果是从列表页传递过来有值则默认选中
      if (this.currentFundsPool.amount) {
        this.nowFundsPool = this.currentFundsPool
        this.finallyResult = this.currentFundsPool.amount + this.currentFundsPool.interest
      }
    }
  }
</script>
