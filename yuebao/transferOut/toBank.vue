<template>
  <div>
    <div class="common-list magt10">
      <dl class="enter-list">
        <dd @click="enabledMultipleBankCard && showPopup()">
          <h2>银行</h2>
          <p>
            <span>{{defaultBank}}</span>
            <!-- <span>北京农商银行(****1234)</span> -->
            <i v-if="enabledMultipleBankCard" class="yo-icon icon-arr-down"></i>
          </p>
        </dd>
      </dl>

      <dl class="input-list">
         <!--余额宝下拉框-->
         <selectList @getMoney="getDrawMoney"></selectList>
            
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
      <!-- <p class="public-tips text-right yow-content magt10">当前余额<em class="star">¥{{balanceMoney | moneyFilter}}元</em></p> -->
    </div>

    <!-- 提交按钮 -->
    <div class="yow" :userBankList='userBankList'>
      <div class="pay-btn">
        <a class="btn  btn-lg btn-block" :class="submitFlag ? 'btn-primary' : 'btn-disabled'" href="javascript:void(0);" @click="submitFlag && submit()">{{buttonMsg}}</a>
      </div>
    </div>

    <!-- 银行卡列表 -->
    <mt-popup v-model="popupVisible" position="bottom">
      <div class="public-pop bank-pop fixed-height">
        <div class="public-pop-title">
          <a href="javascript:void(0);"></a>
          <h3>银行选择</h3>
          <a href="javascript:void(0);"></a>
        </div>
        <dl class="platform-list bank-list bank-color">
          <!-- <dd>中国银行中国银行中国银行中国银行中国(<p>****1234</p>)</dd> -->
          <!-- <dd>招商银行(<p>****1234</p>)</dd> -->
          <dd v-for="bank in userBankCardsList" :key='bank.id' @click="changeBank(bank)">{{bank.bankName}}(<p>****{{bank.bankAccount}}</p>)</dd>
          <dd v-if="isAdd" class="add-bank corr-col" @click="jumpAddBank()"><i class="yo-icon icon-add"></i>添加银行卡</dd>
        </dl>
      </div>
    </mt-popup>
    <card-verify v-if="showVerify && userBankList" :bankInfo='bankInfo' @onShowVerify='checkVerfyBankFn'></card-verify>
    <!--银行卡信息-->
    <bank-detail v-if="showBankDetail" :isDetailPage='isDetailPage' :bankItem='bankItem'></bank-detail>
  </div>
</template>

<script>
import Vue from 'vue'
import Popup from 'mint-ui/lib/popup'
import * as yuebaoType from '@/base/store/yuebao/type'
import initData from '@/base/utils/initData'
import filter from '@/base/utils/filter'
import draw from '@/web/components/memberCenter/payment/draw.js'
import cardVerify from '../../memberCenter/bank/firstCardVerify'
import bankDetail from '../../memberCenter/bank/bankDetail.vue'
import selectList from '@/web/components/yuebao/transferOut/selectList'
import withdrawPwd from '@/base/components/common/withdrawPwd'
Vue.component(Popup.name, Popup)
export default {
  mixins: [initData, filter, draw],
  data () {
    return {
      nowType: 0,
      drawMoney: '', // 取款金额
      drawMoneyPwd: '', // 取款密码
      popupVisible: false, // 充值方式列表
      isOpen: true, // 用于限制点击过快处理
      isAdd: true, // 是否显示添加银行卡按钮
      defaultBank: '', // 默认选择的提款银行
      messageId: '', // 选择的银行id
      showVerify: false,
      balanceMoney: 0,
      isDetailPage: false, // 确定添加银行卡页面类型
      bankItem: {}, // 用来验证银行卡信息是否完整
      bankId: '', // 用来验证银行卡信息是否完整
      bankAccount: '', // 用来验证银行卡信息是否完整
      province: '', // 用来验证银行卡信息是否完整
      city: '', // 用来验证银行卡信息是否完整,
      buttonMsg: '确认转出',
      isInited: false, // 是否已初始化数据
      userBankCardsList: [] // GPO-9234 用于数组循环 默认银行卡排在首位
    }
  },
  watch: {
    popupVisible (val) {
      if (val) { // 显示银行选择列表
        window.indexMain.appHiddenOverflow(true) // 禁用背景滑动
      } else {
        window.indexMain.appHiddenOverflow(false) // 解禁背景滑动
      }
    }
  },
  beforeDestroy () {
    this.$store.state.bindBankcard.userBankList = []
    window.indexMain.appHiddenOverflow(false) // 解禁背景滑动
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
    submit () {
      if (this.defaultBank === '请选择' || this.defaultBank === '') {
        window.layer.msgWarn('请选择银行卡')
        return false
      }
      if (!this.subCheck) return ''
      if (!this.userInfo.realName) {
        window.layer.msgWarn('为保证您的权益，请先绑定您的出款资讯！', () => {
          window.router.push('/wap/index')
        })
        return false
      }
      if (!(!!this.bankId && this.bankId !== -1 && !!this.bankAccount && !!this.province && !!this.city)) {
        window.layer.confirm({content: '该银行卡信息不完整，请完善该卡的出款资讯!', btn: ['去完善', '取消']}, () => {
          this.$store.state.bindBankcard.showBankDetail = true
          this.isDetailPage = true
        }, () => {

        })
        return false
      }
      if (this.isOpen) {
        this.submitRequest()
      }
    },
    submitRequest () {
      this.buttonMsg = '转出中'
      // 如果是余额宝
      let param = {
        'yuebaoFundsPoolId': this.yuebaoFundsPoolId,
        'amount': this.drawMoney,
        'bankType': '0',
        'drawPwd': this.drawMoneyPwd,
        'isyuebao': '1',
        'drawBankId': this.messageId
      }
        // 提交取款申请
      this.$store.dispatch(yuebaoType.DRAW_TO_BANK_ACTION, param).then(res => {
        if (res.status === '1') {
          this.$store.state.yuebao.currentFundsPool = {}
          this.$store.state.yuebao.yuebaoFundsPoolId = null
          window.layer.msgWarn('已提交成功，请等待系统管理员审批！', function () {
            window.router.push('/wap/yuebao/yuebaoHome')
          })
        } else {
          window.layer.msgWarn(res.msg)
        }
      }).catch(res => {
        console.log(' DRAW_MONEY_ACTION store error')
      })
      // 限制两秒才允许再次点击开奖结果
      this.isOpen = false
      setTimeout(() => {
        this.isOpen = true
        this.buttonMsg = '确认转出'
      }, 1500)
    },
    showPopup () {
      this.popupVisible = true
    },
    getDrawMoney (val) {
      this.drawMoney = val
    },
    // 选择银行
    changeBank (bank) {
      console.log(bank)
      this.defaultBank = `${bank.bankName || ''}(****${bank.bankAccount})`
      this.messageId = bank.id
      this.popupVisible = false
      // 用来验证银行卡信息是否完整
      this.bankItem = bank
      this.bankId = bank.bankId
      this.bankAccount = bank.bankAccount
      this.province = bank.province
      this.city = bank.city
    },
    jumpAddBank () {
      if (this.userBankList && this.userBankList.length > 0) {
        if (!this.userInfo.realName) {
          window.layer.msgWarn('真实姓名为空，请联系客服处理!', () => {
            window.router.push('/wap/index')
          })
          return false
        }
        sessionStorage.setItem('switchBank', '/wap/yuebao/transferOut/toBank')
        this.bankInfo = this.userBankList[0]
        this.showVerify = true
        this.popupVisible = false
      } else {
        sessionStorage.setItem('switchBank', '/wap/yuebao/transferOut/toBank')
        window.router.push('/wap/addBank')
      }
    },
    checkVerfyBankFn (data) {
      this.showVerify = data
    }
  },
  computed: {
    yuebaoFundsPoolId () {
      return this.$store.state.yuebao.yuebaoFundsPoolId
    },
    // 是否开启多张银行卡 true为开启了多张银行卡
    enabledMultipleBankCard () {
      return this.$store.state.bindBankcard.enabledMultipleBankCard
    },
    // 获取用户银行卡列表
    userBankList () {
      let userBankList = this.$store.state.bindBankcard.userBankList
      // let bankCards
      this.userBankCardsList = [] // 进入页面先清空数据，否则会出现多条重复数据的问题
      if (userBankList && userBankList.length > 0) {
        // onlineDraw = this.$store.state.payment.onlineDraw
        // userBankList = userBankList
        userBankList.forEach(item => {
          console.log(item)
          if (item.defaulted === 1) { // defaulted=1时为默认银行卡 0或其他值时非默认银行卡,取，默认银行卡的值为进入页面就选中的银行卡
            this.defaultBank = `${item.bankName || ''}(****${item.bankAccount})`
            this.messageId = item.id
            this.bankItem = item
            this.bankId = item.bankId
            this.bankAccount = item.bankAccount
            this.province = item.province
            this.city = item.city
            this.userBankCardsList.unshift(item) // 选则银行卡时 将默认银行卡放在银行卡列表首位
          } else {
            this.userBankCardsList.push(item) // 其他银行卡依次放在后面
          }
        })

        if (userBankList.length >= 5) {
          this.isAdd = false
        } else {
          this.isAdd = true
        }
        return userBankList
      } else {
        this.defaultBank = '请选择'
      }
    },
    place () {
      if ((this.drawMoney === '' || this.drawMoney === null) && this.onlineDraw && this.onlineDraw.quantitySet) {
        return '请输入' + this.onlineDraw.quantitySet.drawLowOnline + '-' + this.onlineDraw.quantitySet.drawUpOnline + '元'
      } else {
        return '请输入提现金额'
      }
    },
    showBankDetail () {
      return this.$store.state.bindBankcard.showBankDetail
    },
    submitFlag () {
      if (this.isInited && this.defaultBank && this.defaultBank !== '请选择' && !!this.drawMoney && !!this.drawMoneyPwd && this.drawMoneyPwd.length >= 4) {
        return true
      } else {
        return false
      }
    }
  },
  created () {
    this.$store.dispatch('getUserBankCards.action', {'loading': 'pass'}).then(res => {
      this.isInited = true // 标记已初始化数据
      let userBankCards = res.data
      if (userBankCards && userBankCards.length <= 0) {
        window.layer.msgWarn('为了保证您的权益，请先绑定您的出款资讯！', () => {
          sessionStorage.setItem('switchBank', '/wap/yuebao/transferOut/toBank')
          localStorage.regok = 1  // 通过状态是12进入绑定银行卡页面的 返回时统一到首页 GPO-8375
          window.router.push('/wap/addBank')
        })
      } else {
        if (!this.userInfo.realName) {
          window.layer.msgWarn('您的信息不完整,不能取款,请联系客服!', () => {

          })
        } else {

        }
      }
    })

    this.$store.state.home.headerType = 3
    this.$store.state.home.headTitle = '提现'
    this.$store.state.home.headLeft = ''
    this.$store.state.home.headRight = 'icon-service'
    this.$store.state.home.headRiTxt = '客服'
  },
  components: {
    cardVerify,
    bankDetail,
    selectList,
    withdrawPwd
  }
}
</script>
