<template>
  <div class="container-fluid admin-header">
    <div class="row">
      <div class="col-md-2 text-center">
        <h2 style="text-align:left">{{store.info}}</h2>
        <h3 style="text-align:left">{{store.name}}<span @click="exit" class="exit">[退出]</span></h3>
      </div>
      <div class="col-md-9">
        <ul class="nav nav-pills navbar-right">
          <li v-if="isHasAuthority"><a v-link="{ path: '/site/order'}" >点单</a></li>
          <li v-if="isHasAuthority"><a v-link="{ path: '/site/member'}">会员</a></li>
          <li v-if="isHasAuthority"><a v-link="{path: '/site/instock' }">库存</a></li>
          <li v-if="isHasAuthority || isLookAuthority"><a v-link="{path: '/site/billing'}">结算</a></li>
          <li v-if="isHasAuthority"><a v-link="{path: '/site/tranquery'}">交易查询</a></li>
          <li v-if="isHasAuthority"><a href='#'>微商城订单</a></li>
          <!--<li><a v-link="{path: '/site/microshoporder'}">微商城订单</a></li>-->
        </ul>
      </div>
      <div class="col-md-1"></div>
    </div>
  </div>
</template>
<script>
  import {storeName,storeAccount,storeInfo,storeAuthority} from '../../publicFunction/index'
  export default{
    name: 'site-nav',
    ready: function () {
      if(storeAuthority.indexOf('1')>-1){
        this.isHasAuthority = true
      }else {
        this.isHasAuthority = false
        this.isLookAuthority = true
      }
    },
    methods: {
      exit: function () {
//         TODO 未来可能有接口，暂时如此
        window.localStorage.setItem('token',null)
        window.localStorage.setItem('storeName',null)
        window.localStorage.setItem('storeAccount',null)
        window.location.href ='#!/site/login'
        window.location.reload()
      }
    },
    data: function () {
      return {
        isHasAuthority: false,
        isLookAuthority: false,
        store: {
          name: storeName,
          account: storeAccount,
          info: storeInfo
        }
      }
    }
  }
</script>
<style scoped>
  a.active{
    color: #000;
    border-top: 6px solid #ff500b;
    border-bottom: 3px solid #c3c3c3;
    background-color: #fff;
    padding-top: 24px;
  }
  .exit{
    cursor: pointer;
  }
</style>
