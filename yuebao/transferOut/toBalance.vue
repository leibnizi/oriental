<template>
    <div class=" magt10">

        <!--余额宝下拉框-->
        <selectList v-on:toAmount="amountFun" @getMoney="getDrawMoney"></selectList>

        <div class="common-list">
          <!-- 申请优惠 -->
          <dl class="enter-list" v-show="discountFun">
            <dd>
              <h2>申请优惠</h2>
              <p>
                <mt-switch v-model="discount" :disabled="applyDiscount" @change="toDiscount()"></mt-switch>
                <span class="discountTips" v-if="isInited">优惠{{offer}}，{{this.quantitySet.codeMultiple}}倍打码</span>
                <span class="discountTips" v-if="!isInited">优惠- -，- -倍打码</span>
              </p>
            </dd>
          </dl>

          <dl class="input-list">

              <dd class="magt10">
                <h2>取款密码</h2>
                <div class="mn-password-input" @click="openPwd">
                  <span v-if="!drawMoneyPwd">请输入取款密码</span>
                  <i v-for="(item, index) in drawMoneyPwd.length" :key="index" class="mn-password-i"></i>
                </div>
                <i v-if="drawMoneyPwd" @click="clearVal()" style="color:#ccc;font-size: .5rem;" class="login-icon-right yo-icon icon-close-hollow"></i>
                <withdrawPwd ref="withdrawPwd" @pwdEnd="pwdEnd" :pwdTitle="'请输入取款密码'"></withdrawPwd>
              </dd>
          </dl>
        </div>

        <!-- 按钮 -->
        <div class="magt10 text-center pay-btn">
          <button class="btn btn-primary btn-lg btn-block" @click="submit()"
                  :class="this.isInited && (this.yuebaoFundsPoolId || this.yuebaoFundsPoolId === 0) && this.drawMoneyPwd.length === 4 && this.drawMoney > 0? 'canClick': 'notClick'"
                  :disabled="!(this.isInited && (this.yuebaoFundsPoolId || this.yuebaoFundsPoolId === 0) && this.drawMoneyPwd.length === 4 && this.drawMoney > 0)"
                  >{{buttonMsg}}</button>
        </div>

    </div>

</template>

<script>
import Vue from 'vue'
import * as type from '@/base/store/yuebao/type'
import selectList from './selectList'
import withdrawPwd from '@/base/components/common/withdrawPwd'
import Switch from 'mint-ui/lib/switch'
Vue.component(Switch.name, Switch)
export default {
  components: {
    selectList,
    withdrawPwd
  },
  data () {
    return {
      drawMoney: '', // 出款金额
      drawMoneyPwd: '', // 出款密码
      buttonMsg: '确认转出',
      isOpen: true, // 用于限制点击过快处理
      offer: 0, // 优惠金额
      ifFlag: false,
      discount: true, // 申请优惠，默认选中申请
      openStroe: false,
      nowFundsPool: {}, // 当前页面选中的参数
      amount: 0,
      isInited: false // 是否已初始化数据
    }
  },
  methods: {
    // 清空取款密码
    clearVal () {
      this.drawMoneyPwd = ''
    },
    // 取款密码
    pwdEnd (val) {
      this.drawMoneyPwd = val
    },
    // 打开密码输入框弹框
    openPwd () {
      this.$refs.withdrawPwd.$emit('openPwd')
    },
    // 初始化函数
    initData () {
      // 查询列表
      const p1 = this.$store.dispatch(type.QUERY_FUNDSPOOL_LIST_ACTION, { 'loading': 'pass' })
      // 查询存取款设定
      const p2 = this.$store.dispatch(type.INIT_PAGE_ACTION, { 'loading': 'pass' })
      Promise.all([p1, p2]).then((result) => { // 所有请求响应成功之后执行
        this.isInited = true // 标记已初始化数据
      })
    },
    /**
    * 数字格式化 2位
    * author: adaf
    */
    round2 (v) {
      let d = parseFloat(v)
      if (isNaN(d)) d = 0.0
      d = Math.floor(d * 100) / 100 // 强制截取2位小数
      return d.toFixed(2) // 保留2位小数
    },
    /**
     * 监听子组件的金额传输
     */
    amountFun (amount) {
      this.amount = amount
      // 优惠信息提示
      let money = parseFloat(this.amount)
      // 计算优惠金额
      let yhMoney = money * parseFloat(this.quantitySet.percentageDiscount) / 100
      // 已经优惠次数
      let haveDiscountTimes = this.quantitySet.haveDiscountTimes
      // 优惠次数 类型 0 -首次 1 -每次
      let discountTimes = this.quantitySet.discountTimes
      // 稽核优惠标准
      let auditDiscountStandard = this.quantitySet.auditDiscountStandard
      // 优惠系数 -次数
      let preRatioOnline = 0
      // 从记录跳转过来会报错
      try {
        preRatioOnline = this.quantitySet.quantitySet.preRatioYuebao
      } catch (e) {
      }
      // 优惠上限
      let preUp = this.quantitySet.preUp
      // 优惠
      if (preUp <= yhMoney) {
        yhMoney = preUp
      }
      if (discountTimes === 0 || discountTimes === '0') {
        // 判断如果后台设置的是首次存款才有优惠时的处理
        if (haveDiscountTimes === 0 || haveDiscountTimes === '0') {
          if (parseFloat(money) >= auditDiscountStandard) {
            yhMoney = money * parseFloat(this.quantitySet.percentageDiscount) / 100
          } else {
            yhMoney = 0
          }
        } else {
          yhMoney = 0
        }
      } else if (discountTimes === 1 || discountTimes === '1') {
        if (parseFloat(money) >= auditDiscountStandard) {
          yhMoney = money * parseFloat(this.quantitySet.percentageDiscount) / 100
        } else {
          yhMoney = 0
        }
      } else if (discountTimes === 3 || discountTimes === '3' ||
        discountTimes === 4 || discountTimes === '4') {
        // 每次都有优惠
        if (parseFloat(money) >= auditDiscountStandard) {
          // 次数小于优惠系数
          let times = haveDiscountTimes
          // 每天前N次
          if (discountTimes === 4 || discountTimes === '4') {
            times = this.quantitySet.haveDayDiscountTimes
          }
          // 次数小于优惠系数
          if (preRatioOnline >= parseInt(times) + 1) {
            yhMoney = money * parseFloat(this.quantitySet.percentageDiscount) / 100
          } else {
            yhMoney = 0
          }
        } else {
          yhMoney = 0
        }
      }
      if (preUp <= yhMoney) {
        yhMoney = preUp
      }
      if (yhMoney >= 0) {
        this.offer = this.round2(yhMoney)
      }
    },
    assertdrawMoneyPwd () {
      this.drawMoneyPwd = this.drawMoneyPwd.replace(/[^0-9]/ig, '')
    },
    getDrawMoney (val) {
      this.drawMoney = val
    },
    /**
     * 0：不申请优惠，1：申请优惠
     */
    toDiscount () { // 向父组件传输是否选中优惠
      this.$emit('toDiscount', this.discount ? 1 : 0)
    },
    submit () {
      if (!this.isInited) {
        return
      }
      if (this.drawMoneyPwd === '' || this.drawMoneyPwd === null) {
        window.layer.msgWarn('密码不可以为空！')
        return false
      }
      if (!this.drawMoney) {
        window.layer.msgWarn('请选择转出金额!')
        return false
      }
      if (this.drawMoneyPwd && (this.drawMoneyPwd + '').length !== 4) {
        window.layer.msgWarn('取款密码为4位数字！')
        return false
      }
      if (this.isOpen) {
        this.buttonMsg = '转出中'
        this.submitRequest()
      }
    },
    submitRequest () {
      let param = {
        drawPwd: this.drawMoneyPwd, // 取款密码
        yuebaoFundsPoolId: this.yuebaoFundsPoolId,
        hasDiscount: Number(this.discount), // 是否优惠，0不申请，1申请
        'isyuebao': '1'
      }
      // 提交取款申请
      this.$store.dispatch(type.DRAW_TO_BALANCE_ACTION, param).then(res => {
        if (res.status === '1') {
          this.$store.state.yuebao.currentFundsPool = {}
          this.$store.state.yuebao.yuebaoFundsPoolId = null
          window.layer.msgWarn('转出成功，请耐心等待！', function () {
            window.router.push('/wap/yuebao/yuebaoHome')
          })
        }
      }).catch(res => {
        console.log(' DRAW_TO_BALANCE_ACTION store error')
      })
      // 限制两秒才允许再次点击开奖结果
      this.isOpen = false
      setTimeout(() => {
        this.isOpen = true
        this.buttonMsg = '确认转出'
        this.initData()
      }, 1500)
    }
  },
  computed: {
    type () {
      return this.$store.state.home.type
    },
    /**
     * 用于判断是否可以放弃优惠
     */
    applyDiscount () {
      // 0:不可放弃
      let preDepositStatusOnline = this.quantitySet.preDepositStatusOnline
      if (preDepositStatusOnline === '0') {
        // 不可放弃优惠 默认选中优惠 不可选
        return true
      } else {
        return false
      }
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
    yuebaoFundsPoolId () {
      return this.$store.state.yuebao.yuebaoFundsPoolId
    },
    quantitySet () {
      return this.$store.state.yuebao.quantitySet
    },
    /**
     * 计算是否显示优惠
     */
    discountFun () {
      if (this.quantitySet.discountTimes) {
        // 判断是否首次优惠，如果是首次优惠则根据用户存款次数显示优惠框，否则不显示优惠框
        let discountTimes = this.quantitySet.discountTimes
        // 用户已存款次数
        let haveDiscountTimes = this.quantitySet.haveDiscountTimes
        // 用户每天的存款次数
        let haveDayDiscountTimes = this.quantitySet.haveDayDiscountTimes
        // 代理后台设置的可优惠次数
        let preRatioOnline = 0
        // 从记录跳转过来会报错
        try {
          preRatioOnline = this.quantitySet.quantitySet.preRatioYuebao
        } catch (e) {
        }
        // discountTimes: 0 首次，1 每次，2 不优惠，3 前N次，4 每天前N次（原有字段）
        if (discountTimes === 0 || discountTimes === '0') {
          // 判断如果后台设置的是首次存款才有优惠时的处理
          if (haveDiscountTimes !== '0') {
            // 隐藏优惠
            return false
          } else {
            return true
          }
        } else if (discountTimes === 1 || discountTimes === '1') { // 每次
          return true
        } else if (discountTimes === 2 || discountTimes === '2') { // 不优惠
          return false
        } else if (discountTimes === 3 || discountTimes === '3') { // 前N次
          // 非首次
          if (preRatioOnline < parseInt(haveDiscountTimes) + 1) {
            // 隐藏优惠
            return false
          } else {
            return true
          }
        } else if (discountTimes === 4 || discountTimes === '4') { // 每天前N次
          // 非首次
          if (preRatioOnline < parseInt(haveDayDiscountTimes) + 1) {
            // 隐藏优惠
            return false
          } else {
            return true
          }
        }
      }
    }
  },
  created () {
    if (Number(this.type) === 1) {
      window.layer.msgWarn('该功能只对正式账号进行开放，请您注册正式账号！')
      this.$router.push('/wap/index')
    }
    if (this.$store.state.yuebao.currentFundsPool.amount) {
      this.amount = this.$store.state.yuebao.currentFundsPool.amount
      this.drawMoney = this.$store.state.yuebao.currentFundsPool.amount
      // 计算优惠
      this.amountFun(this.drawMoney)
    }
    this.initData()
  }
}
</script>
