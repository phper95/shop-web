/**
    付款金额组件
    传参arr
    示例:
    arr: [
        ...
        'count': 100,   //总计
        'postage': 10,  //邮费
        'activity': 40,
        'ticket': 40,
        ...
    ]
 */
<template>
  <div class="shouldPay">
      <div class="priceCom">商品合计：<span>￥{{ obj.total_price }}</span></div>
      <div class="priceCom">邮费：<span>￥{{ obj.total_postage }}</span></div>
      <!-- <div class="priceCom">活动优惠：<span>-￥{{ obj.deduction_price }}</span></div> -->
      <div class="priceCom">优惠卷：<span>-￥{{ obj.coupon_price }}</span></div>
      <div class="priceCom count">最终金额：<span>￥{{ shouldPay }}</span></div>
  </div>
</template>

<script>
export default {
    name: 'shouldPay',
    props: {
        obj: Object
    },
    computed: {
        shouldPay () {
            return this.obj.total_price + this.obj.total_postage - this.obj.deduction_price - this.obj.coupon_price
        }
    }
}
</script>

<style lang="scss">
.shouldPay{
    text-align: right;
    margin: 30px;
    .priceCom{
        font-size: 16px;
        color: #999999;
        margin-bottom: 18px;
        span{
            color: #F26E47;
        }
    }
    .count{
        padding: 15px 0;
        font-size: 18px;
        span{
            font-size: 30px;
        }
    }
}
</style>
