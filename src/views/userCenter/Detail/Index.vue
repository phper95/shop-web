/**
    订单详情页
 */
<template>
  <div class="details">
        <nav>
          <div>订单详情</div>
          <div class="dealBtn">
              <div class="order_id">订单号：{{ this.$route.query.id }}</div>
              <div class="orderBtn">
                  <button
                  v-for="(item, index) in this.layoutButton"
                  :key="index"
                  :class="item.type"
                  @click="statusBtn(item.interface)"
                  >{{ item.des }}</button>
              </div>
          </div>
        </nav>
        <!-- 组件部分，根据状态展示 -->
        <section v-loading="loading">
            <OrderStatus :status="this.orderdata[0].status"/>
            <Combination class="common"
            v-if="this.orderdata[0].status === 11"
            :pid="this.orderdata[0].pink_id"
            @getInviPoster="isGetPoster"/>
            <Commod
            class="common"
            :listarr="this.orderdata[0].cart_info"/>
            <Recipient class="common"
            :obj="this.orderdata[0].address"/>
            <ExpressCom
            class="common"
            v-if="this.orderdata[0].status !== 0 && this.orderdata[0].status !== 10 && this.orderdata[0].status !== 11 && this.orderdata[0].status !== 12"
            :obj="this.orderdata[0].express"/>
            <ShouldPay :obj="this.orderdata[0].money"/>
        </section>
        <!-- 删除订单 -->
        <el-dialog
        :visible.sync="delDialogVisible"
        width="600px"
        center>
            <div class="diaContent">
                <div class="diaTitle">确认删除{{ this.$route.query.id }}订单吗？</div>
            </div>
            <div class="diaBtn">
                <button @click="delDialogVisible = false">取 消</button>
                <button class="yes" @click="dleOrder(orderdata[0].order_id)">确 定</button>
            </div>
        </el-dialog>
        <!-- 取消订单 -->
        <el-dialog
        :visible.sync="cancelDialogVisible"
        width="600px"
        center>
            <div class="diaContent">
                <div class="diaTitle">确认取消订单吗？</div>
            </div>
            <div class="diaBtn">
                <button @click="cancelDialogVisible = false">取 消</button>
                <button class="yes" @click="canOrder(orderdata[0].order_id)">确 定</button>
            </div>
        </el-dialog>
        <!-- 邀请好友 -->
        <el-dialog
        class="delDia"
        :visible.sync="inviteDialogVisible"
        width="670px"
        center>
            <div class="diaContent noBottom">
                <div class="diaTitle">邀请好友</div>
                <div class="diaText">扫描下方二维码或复制链接分享给好友吧</div>
                <el-image class="pink" :src="this.qrcode"></el-image>
                <!-- <VueQrcode
                class="qrcode"
                :width=270
                :margin=1
                :value="this.qrcode" /> -->
            </div>
            <div class="diaBtn">
                <!-- <span class="shareTxt">{{ this.qrcode }}</span> -->
                <button class="cpqr" @click="useClipboard(qrcode,$event)">复制链接</button>
            </div>
        </el-dialog>
        <!-- 确认收货 -->
        <el-dialog
        :visible.sync="receDialogVisible"
        width="600px"
        center>
            <div class="diaContent">
                <div class="diaTitle">确认收到货了吗</div>
                <div class="diaText">为保障您的售后权益，请核对商品无误后确认收货</div>
            </div>
            <div class="diaBtn">
                <button @click="receDialogVisible = false">取 消</button>
                <button class="yes" @click="takOrder(orderdata[0].order_id)">确 定</button>
            </div>
        </el-dialog>
  </div>
</template>

<script>
/**
 * 商品信息组件 Commod
 * 收件信息组件 Recipient
 * 付款金额组件 ShouldPay
 * 订单状态组件 OrderStatus
 * 拼团组件 Combination
 * 二维码插件 VueQrcode
 */
import Commod from '../components/commodity.vue'
import Recipient from '../components/recipient'
import ShouldPay from '../components/shouldPay'
import OrderStatus from '../components/status'
import ExpressCom from '../components/express'

import VueQrcode from 'vue-qrcode'
import Clipboard from 'clipboard'

import {
  cancelOrder,
  deleteOrder,
  detailOrder,
  takeOrder
} from '@/api/order.js'
import ButtonList from './data'

export default {
    components: {
        Commod,
        Recipient,
        ShouldPay,
        OrderStatus,
        ExpressCom,
        VueQrcode
    },
    created () {
        this.detaOrder()
    },
    data () {
        return {
            arr: [
                {
                    'status': 0,
                    'ordertime': '2020-10-12 12:12:00',
                    'order_id': '201545000',
                    'payRemain': '58:55:66',
                    'content': [
                        {
                            'status': 0,
                            'img': '',
                            'title': 'EMPORIO ARMANI刺绣LOGO连帽卫衣',
                            'color': '灰色',
                            'size': 'M',
                            'sum': '1',
                            'price': '500',
                            'url': ''
                        },
                        {
                            'status': 0,
                            'img': '',
                            'title': 'EMPORIO ARMANI刺绣LOGO连帽卫衣',
                            'color': '灰色',
                            'size': 'XXL',
                            'sum': '1',
                            'price': '500',
                            'url': ''
                        }
                    ],
                    'address': {
                        'name': '',
                        'phoneNum': '',
                        'address': ''
                    }
                }
            ],
            loading: false,
            orderdata: [
                {
                    'status': null,
                    'cart_info': [],
                    'money': {},
                    'address': {}
                }
            ],
            layoutButton: [],
            delDialogVisible: false,
            cancelDialogVisible: false,
            inviteDialogVisible: false,
            receDialogVisible: false,
            qrcode: '' // 邀请好友海报
        }
    },
    methods: {
        // 订单详情
        async detaOrder () {
            this.loading = true
            if (this.$route.query.id.length === 0 || !this.$route.query.id) { return }
            var res = await detailOrder(this.$route.query.id)
            if (res.status !== 200) {
                this.$message.error(res.msg || '网络异常！')
                return
            }
            this.orderdata = this.getObjData(res)
            if (this.orderdata[0].pink_id !== 0) { // 是否拼团
                var pinkStatus = await combinationpink(this.orderdata[0].pink_id)
                if (pinkStatus.data.pinkBool === -1) {
                    this.orderdata[0].status = 12 // 拼团失败状态码
                }
            }
            if (this.orderdata[0].cart_info[0].is_reply !== 0) {
                this.orderdata[0].status = 4
            }
            this.selectBtn(this.orderdata[0].status)
            this.loading = false
        },
        // 解析单个订单数据
        getObjData (res) {
            if (!res) { return }
            if (res.data.paid !== 1) { // 已支付为1
                // 未支付
                res.data.status = 10
            }
            // if (res.data.paid === 1 && res.data.pink_id !== 0) {
            //     // 拼团订单
            //     res.data.status = 11
            // }
            return [{
                'status': res.data.status,
                'unique': res.data.unique,
                'id': res.data.id,
                'create_time': res.data.create_time,
                'order_id': res.data.order_id,
                'display_order_id': res.data.display_order_id,
                'paid': res.data.paid,
                'pay_time': res.data.pay_time,
                'pay_price': res.data.pay_price,
                'pink_id': res.data.pink_id,
                'combination_id': res.data.combination_id,
                'cart_info': res.data.cart_info.map(i => {
                    return {
                    'cart_id': i.id,
                    'product_info_id': i.product_info.id,
                    'attrid': i.product_info.attr_info.id,
                    'combination_id': i.combination_id,
                    'product_id': i.product_id,
                    'attr_product_id': i.product_info.attr_info.product_id,
                    'cart_num': i.cart_num,
                    'is_reply': i.is_reply, // 是否评价
                    'true_price': i.true_price,
                    'store_name': i.product_info.store_name,
                    'img': i.product_info.attr_info.image,
                    'sku': i.product_info.attr_info.sku
                    }
                }),
                'address': {
                    'real_name': res.data.real_name,
                    'user_phone': res.data.user_phone,
                    'user_address': res.data.user_address
                },
                'money': {
                    'total_price': res.data.total_price, // 订单总价
                    'total_postage': res.data.total_postage, // 总邮费
                    'coupon_price': res.data.coupon_price, // 优惠卷金额
                    'deduction_price': res.data.deduction_price, // 抵扣金额
                    'pay_price': res.data.pay_price
                },
                'express': {
                    'orderCode': res.data.order_id,
                    'delivery_name': res.data.delivery_name,
                    'delivery_id': res.data.delivery_id,
                    'logisticCode': res.data.delivery_id,
                    'delivery_sn': res.data.delivery_sn
                }
            }]
        },
        // 根据订单的状态码来进行按钮渲染
        selectBtn (status) {
            this.layoutButton = ButtonList.buttonlist.map(item => {
                if (item.status.indexOf(status) !== -1) {
                    return item
                }
            }).filter(r => r)
        },
        // 状态按钮
        statusBtn (inte) {
            console.log(inte)
            switch (inte) {
                case 'cancelOrder':
                    this.cancelDialogVisible = true
                    break
                case 'payNow':
                    console.log('立即付款')
                    break
                case 'getPayBack':
                    console.log('申请退款')
                    // 跳转售后
                    if (!this.$route.query.id) { return }
                    this.NavTo(
                        '/submitAS',
                        {'id': this.$route.query.id}
                    )
                    break
                case 'inviCombination':
                    this.inviteDialogVisible = true
                    this.getInviPoster(this.orderdata[0].pink_id)
                    break
                case 'cancelPayBack':
                    console.log('取消退款')
                    break
                case 'deleteOrder':
                    console.log('删除订单')
                    this.delDialogVisible = true
                    break
                case 'againCombination':
                    console.log('再次开团')
                    break
                case 'remindShip':
                    console.log('提醒发货')
                    break
                case 'sureReceive':
                    console.log('确认收货')
                    this.receDialogVisible = true
                    break
                case 'subAssess':
                    // var isReplay = 0
                    // this.orderdata[0].cart_info.forEach(i => {
                    //     if (i.isReply !== 0) {
                    //         isReplay++
                    //     }
                    // })
                    // console.log(isReplay, this.orderdata[0].cart_info.length)
                    // if (isReplay === this.orderdata[0].cart_info.length) {
                    //     this.$message.error('所有商品均已评论过')
                    //     return
                    // }
                    if (this.orderdata[0].cart_info[0].is_reply !== 0) {
                        this.$message.error('该订单已评价过')
                        return
                    }
                    this.NavTo('/evaluate', {'id': this.orderdata[0].unique})
                    break
                case 'buyAgain':
                    console.log('再次购买')
                    // 再次购买  订单详情中商品id数量大于1 不显示
                    // 传参商品详情页路径、商品id
                    this.NavTo(
                        '/',
                        {'product_id': ''}
                    )
                    break
                default:
                    this.$message.error('选项错误')
                    console.error('无匹配选项')
                    break
            }
        },
        // 取消订单
        canOrder (id) {
            cancelOrder({'id': id})
            .then(res => {
                if (res.status === 200) {
                    this.$message.success('取消订单成功')
                    // location.reload()
                    this.$router.push('/myOrder')
                } else {
                    this.$message.error(res.msg || '网络异常！')
                }
            })
        },
        // 删除订单
        async dleOrder (id) {
            var res = await deleteOrder({ 'uni': id })
            if (res.status === 200) {
                this.$message.success('删除订单成功')
                location.reload()
            } else {
                this.$message.error(res.msg || '订单尚未完成，无法删除！')
            }
        },
        // 拼团二维码数据获取
        async getInviPoster (id) {
            var res = await getPoster({ 'from': 'h5', 'id': `${id}` })
            if (res.status === 200) {
                this.qrcode = res.data.url
            } else {
                this.$message.error(res.msg)
            }
        },
        NavTo (path, query) {
            this.$router.push({
                'path': path,
                'query': query
            })
        },
        isGetPoster (val) {
            this.inviteDialogVisible = true
            this.getInviPoster(val)
        },
        // 一键复制
        useClipboard (text, event) {
            var clipboard = new Clipboard(event.target, {text: () => text})
            clipboard.on('success', e => {
                this.$message.success('复制成功')
                clipboard.destroy()
            })
            clipboard.on('error', e => {
                this.$message.error('该浏览器不支持自动复制')
                clipboard.destroy()
            })
        },
        // 确认收货
        takOrder (order_id) {
            takeOrder({'uni': order_id}).then(res => {
                if (res.status === 200) {
                this.$message.success('收货成功')
                this.receDialogVisible = false
                location.reload()
                }
            })
        }
    }
}
</script>

<style lang="scss">
.details{
    background-color: #FFF;
    nav{
        margin: 0 30px;
        font-size: 30px;
        color: #333333;
        padding: 30px 0;
        border-bottom: 1px solid #CCCCCC;
        .dealBtn{
            margin-top: 10px;
            height: 40px;
            line-height: 40px;
            font-size: 20px;
            div{
                display: inline-block;
            }
            .orderBtn{
                float: right;
                button{
                    width: 120px;
                    height: 40px;
                    font-size: 15px;
                    cursor: pointer;
                    margin-left: 20px;
                }
                .special{
                    float: right;
                    color: #FFFFFF;
                    background-color: #F26E47;
                }
                .normal{
                    color: #333333;
                    background-color: #FFFFFF;
                    border: 1px solid #333333;
                    &:hover{
                        color: #FFFFFF;
                        background-color: #333333;
                        border: 1px solid #FFFFFF;
                    }
                }
            }
        }
    }
    section{
        .common{
            margin: 0 30px;
            padding: 30px 0;
            border-bottom: 1px solid #CCCCCC;
            .title{
                font-size: 20px;
                color: #323232;
                margin-bottom: 30px;
            }
            .content{
                margin: 10px 0;
                font-size: 16px;
                color: #999999;
                span{
                    color: #323232;
                }
            }
        }
    }
    .noBottom{
     margin: 45px 0 0 0 !important;
    }
    .diaContent{
        margin: 45px 0 70px 0;
        text-align: center;
        .diaTitle{
            font-size: 24px;
            color: #333333;
        }
        .diaText{
            margin-top: 24px;
            font-size: 20px;
            color: #666666;
        }
        .qrcode{
            margin-top: 30px;
        }
        .pink{
          width: 310px;
          height: 600px;
        }
    }
    .diaBtn{
        border-top: 1px solid #000;
        padding-top: 40px;
        button{
            width: 190px;
            height: 60px;
            cursor: pointer;
            font-size: 20px;
            margin-bottom: 10px;
            color: #333333;
            background-color: #FFFFFF;
            border: 1px solid #333333;
        }
        .yes{
            float: right;
            color: #FFFFFF;
            background-color: #333333;
            border: 1px solid #333333;
        }
        .shareTxt{
          display: inline-block;
          width: 340px;
          height: 60px;
          line-height: 60px;
          padding: 0 82px;
          font-size: 20px;
          color: #000000;
        }
        .cpqr{
          display: inline-block;
          width: 100%;
          &:hover{
            color: #FFFFFF;
            background-color: #333333;
            border: 1px solid #333333;
          }
        }
    }
}
</style>
