/**
    收件人信息组件
 */
<template>
<div class="recipient">
    <div class="title">收货信息</div>
    <div class="content">收货人: <span>{{ obj.real_name }}</span></div>
    <div class="content">手机号: <span>{{ obj.user_phone }}</span></div>
    <div class="content">地&nbsp;&nbsp;&nbsp;&nbsp;址: <span>{{ obj.user_address }}</span></div>
</div>
</template>

<script>
export default {
    name: 'recipient',
    props: {
        obj: Object
    }
}
</script>
