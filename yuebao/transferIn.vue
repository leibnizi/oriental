<template>
    <div class="pane" >
      <!-- 头部 -->
      <div class="public-header">
        <div class="common-header hea-default-col">
            <div class="header-content">
                <div class="head-left" @click="goBack()">
                    <i class="yo-icon icon-arr-left"></i>
                </div>
                <div class="head-title">转入余额宝</div>
                <div class="head-right">
                </div>
            </div>
        </div>
      </div>

      <!-- 可滑动区 -->
      <div class="common-main main-box">
        <div class="common-list">
            <dl class="input-list magt10">
              <dd>
                <h2>转入金额(¥)</h2>
                <input class="typeinput input-one input-content" v-model="drawMoney" type="tel" :placeholder="place" @input="drawMoneyInput">
                <a class="withdrawal-txt corr-col" href="javascript:void(0);" @click="isInited && allTakeOut()">全部转入</a>
              </dd>
              <dd>
                <h2>取款密码</h2>
                <div class="mn-password-input" @click="openPwd">
                  <span v-if="!drawMoneyPwd">请输入取款密码</span>
                  <i v-for="(item, index) in drawMoneyPwd.length" :key="index" class="mn-password-i"></i>
                </div>
                <i v-if="drawMoneyPwd" @click="clearVal()" style="color:#ccc;font-size: .5rem;" class="login-icon-right yo-icon icon-close-hollow"></i>
                <withdrawPwd ref="withdrawPwd" @pwdEnd="pwdEnd" :pwdTitle="'请输入取款密码'"></withdrawPwd>
              </dd>
            </dl>
            <p class="public-tips text-right yow-content magt10" style="height: .8rem;color:#666666;">当前余额
              <em class="star" v-if="isInited">¥{{this.onlineDraw.money}}元</em>
              <em class="star" v-if="!isInited"><span style="display: inline;font-size: .4rem;"> -- </span>元</em>
            </p>

            <!-- 按钮 -->
            <div class="yow">
              <div class="pay-btn">
                <a class="btn btn-lg btn-block" :class="submitFlag ? 'btn-primary' : 'btn-disabled'" href="javascript:void(0);" @click="submitFlag && submit()">{{buttonMsg}}</a>
              </div>
            </div>
        </div>
      </div>
  </div>
</template>

<script>
import * as type from '@/base/store/payment/type'
import * as yuebaoType from '@/base/store/yuebao/type'
import * as statusCode from '@/base/utils/status_const'
import withdrawPwd from '@/base/components/common/withdrawPwd'
export default {
  data () {
    return {
      drawMoney: '', // 出款金额
      drawMoneyPwd: '', // 出款密码
      buttonMsg: '确认转入',
      isInited: false, // 是否已初始化数据
      isOpen: true // 用于限制点击过快处理
    }
  },
  components: {
    withdrawPwd
  },
  computed: {
    toTransferPage () {
      return this.$store.state.yuebao.toTransferPage
    },
    submitFlag () {
      if (this.isInited && this.drawMoney && this.drawMoney > 0 && this.drawMoneyPwd.length >= 4) {
        return true
      } else {
        return false
      }
    },
    onlineDraw () {
      return this.$store.state.payment.onlineDraw
    },
    type () {
      return this.$store.state.home.type
    },
    place () {
      if (this.drawMoney === '' || this.drawMoney === null) {
        if (this.onlineDraw && this.onlineDraw.quantitySet) {
          return '请输入' + this.onlineDraw.quantitySet.drawLowOnline + '-' + this.onlineDraw.quantitySet.drawUpOnline + '元'
        }
      } else {
        return ''
      }
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
    goBack () {
      if (this.toTransferPage === 0) {
        this.$router.push('/wap/yuebao/yuebaoHome')
      } else {
        this.$router.push('/wap/yuebao/yuebaoList')
      }
    },
    // 初始化数据
    initDrawData () {
      // 非空时不做重复初始化
      if (window.$.isEmptyObject(this.$store.state.payment.onlineDraw)) {
        this.$store.dispatch(type.INIT_DRAW_DATA_ACTION, { bankType: 0, 'isyuebao': '1', 'loading': 'pass' }).then(res => {
          if (res.status === statusCode.statusConst.FAILURE) { // 失败
            window.router.push('/wap/basicsInfo')
          }
          this.isInited = true // 标记已初始化数据
        })
      } else {
        this.isInited = true // 标记已初始化数据
      }
    },
    // 全部提现功能
    allTakeOut () {
      // 如果余额大于最大可提现金额，直接取最大可提现金额，否则取余额
      if (this.onlineDraw.money > this.onlineDraw.quantitySet.drawUpOnline) {
        this.drawMoney = this.onlineDraw.quantitySet.drawUpOnline
      } else {
        this.drawMoney = parseInt(this.onlineDraw.money)
      }
      // 金额太小不允许提款
      if (this.drawMoney < this.onlineDraw.quantitySet.drawLowOnline) {
        window.layer.msgWarn('您的余额小于最低提款金额，去多多盈利再来提款吧！')
        return false
      }
    },
    /**
     * 出款输入框验证
     */
    drawMoneyInput () {
      if (this.drawMoney === '' || this.drawMoney === null) {
        return
      }
      this.drawMoney = this.drawMoney.replace(/[^0-9]+/g, '')
      if (this.drawMoney < 0) {
        this.drawMoney = 0
      } else {
        if (this.drawMoney !== '') {
          this.drawMoney = Number(this.drawMoney)
        }
        if (this.drawMoney > this.onlineDraw.quantitySet.drawUpOnline) {
          this.drawMoney = this.onlineDraw.quantitySet.drawUpOnline
        }
      }
    },
    assertdrawMoneyPwd () {
      this.drawMoneyPwd = this.drawMoneyPwd.replace(/[^0-9]/ig, '')
    },
    moneyInput () {
      // 金额太小不允许提款
      if (this.drawMoney < this.onlineDraw.quantitySet.drawLowOnline) {
        window.layer.msgWarn('您的余额小于最低提款金额，去多多盈利再来提款吧！')
        return false
      }
    },
    submit () {
      if (this.drawMoney > this.onlineDraw.quantitySet.drawUpOnline) {
        window.layer.msgWarn('最高提款金额为：' + this.onlineDraw.quantitySet.drawUpOnline)
        return false
      }
      if (this.drawMoney < this.onlineDraw.quantitySet.drawLowOnline) {
        window.layer.msgWarn('最低提款金额为：' + this.onlineDraw.quantitySet.drawLowOnline)
        return false
      }
      if (this.drawMoneyPwd === '' || this.drawMoneyPwd === null) {
        window.layer.msgWarn('密码不可以为空！')
        return false
      }
      if (this.drawMoneyPwd && (this.drawMoneyPwd + '').length !== 4) {
        window.layer.msgWarn('取款密码为4位数字！')
        return false
      }
      if (this.isOpen) {
        this.buttonMsg = '提交中'
        // 判断如果有手续费，需要提示给客户在做提款操作
        let sxh = this.onlineDraw.quantitySet.moneyDraw1
        if (sxh > 0 && this.type !== '2') {
          let repeatTimeNumDraw = this.onlineDraw.quantitySet.repeatTimeNumDraw
          let freeTimesDraw = this.onlineDraw.quantitySet.freeTimesDraw
          var msg = '您好，由于您在' + repeatTimeNumDraw + '小时超出免收出款手续费' + freeTimesDraw + '次，本次出款将收取您取款手续费' + sxh + '元！'
          window.layer.confirm(msg, () => {
            this.submitRequest()
          }, () => {
            this.buttonMsg = '提交'
          })
        } else {
          this.submitRequest()
        }
      }
    },
    submitRequest () {
      let param = {
        drawPwd: this.drawMoneyPwd, // 取款密码
        currentTime: '',
        bankType: '0',
        'isyuebao': '1',
        cash: this.drawMoney // 取款金额
      }
      // 提交取款申请
      this.$store.dispatch(yuebaoType.SAVE_YUEBAO_DEPOSIT_ACTION, param).then(res => {
        if (res.status === '1') {
          let cueDrawCheck = this.onlineDraw.quantitySet.cueDrawCheck
          let moneyDrawCheck = this.onlineDraw.quantitySet.moneyDrawCheck
          let timeNumDrawCheck = this.onlineDraw.quantitySet.timeNumDrawCheck
          if (Number(cueDrawCheck) === 1 && this.drawMoney >= moneyDrawCheck && this.type !== '2') {
            let content = '您提款的金额超过' + moneyDrawCheck + '需要审核，将于' + timeNumDrawCheck + '小时内到账，请耐心等候！'
            window.layer.msgWarn(content, function () {
              window.router.push('/wap/yuebao/yuebaoHome')
            })
          } else {
            window.layer.msgWarn(res.msg, function () {
              window.router.push('/wap/yuebao/yuebaoHome')
            })
          }
        }
      }).catch(res => {
        console.log(' DRAW_MONEY_ACTION store error')
      })
      // 限制两秒才允许再次点击开奖结果
      this.isOpen = false
      setTimeout(() => {
        this.isOpen = true
        this.buttonMsg = '提交'
        this.initDrawData()
      }, 1500)
    }
  },
  mounted () {
    this.$store.state.teamRecord.openRecordBox = 0
    if (Number(this.type) === 1) {
      window.layer.msgWarn('该功能只对正式账号进行开放，请您注册正式账号！')
      this.$router.push('/wap/index')
    }
    this.initDrawData()
  }
}
</script>

