<template>
  <site-nav></site-nav>
  <div class="container-fluid">
    <!-- 路径导航 -->
    <ol class="breadcrumb">
      <li class="active"><span class="glyphicon glyphicon-home c-erp" aria-hidden="true"></span> 您当前的位置：库存首页</li>
      <li class="active">收货单汇总</li>
    </ol>

    <!-- 页头 -->
    <div class="page-header">
      <form class="form-inline">
        <div class="form-group">
          <label>单号</label>
          <input type="text" class="form-control" placeholder="" v-model="orderNumber">
        </div>
        <div class="form-group ml10">
          <label>状态</label>
          <select class="form-control" v-model="selectedCheck">
            <option value="">请选择</option>
            <option value="0">未审核</option>
            <option value="1">已审核</option>
          </select>
        </div>
        <div class="form-group ml10">
          <label>制单人</label>
          <select class="form-control" v-model="createPersonId">
            <option value="">请选择</option>
            <option v-for="item in creators" :value="item.id">{{item.name}}</option>
          </select>
        </div>
        <div class="form-group ml10">
          <label>制单时间段</label>
          <date-picker
            :value.sync=query.start_time1"
          >
          </date-picker>
          -
          <date-picker
            :value.sync="query.end_time1"
          >
          </date-picker>
        </div>
        <div class="form-group ml10">
          <label>送货时间段</label>
          <date-picker
            :value.sync="query.end_time1"
          >
          </date-picker>
          -
          <date-picker
            :value.sync="query.end_time2"
          >
          </date-picker>
        </div>

        <span class="btn btn-info" @click="search">搜索</span>
        <a v-link="{ path: '/site/instock/Allot'}"><span class="btn btn-primary fr">新建收货单</span></a>
      </form>
    </div>
    <!--表格 -->
    <summary
      :table-header="gridColumns"
      :table-data="list"
      :page="page"
      :check-url="checkUrl"
      :has-validate-authority="true"
      :has-look-authority = "true"
      :has-finish-authority="true"
      :has-delete-authority= "true"
    ></summary>
  </div>
</template>
<script>
  import $ from 'jquery'
  import SiteNav from '../SiteNav'
  import Grid from '../../common/Grid'
  import Page from '../../common/Page'
  import Summary from  '../../common/Summary'
  import DatePicker from '../../common/DatePicker'
  import {
    requestUrl,
    token,
    searchRequest,
    exchangeData,
    deleteRequest,
    checkRequest,
    finishRequest,
    error,
    changeStatus,
    getDataFromSiteApi
  } from '../../../publicFunction/index'
  export default {
    components: {
      Grid: Grid,
      Page: Page,
      Summary: Summary,
      DatePicker: DatePicker,
      SiteNav: SiteNav
    },
    events: {
//     绑定翻页事件
      pagechange: function (currentpage) {
        this.listData(currentpage)
      },
//     删除请求
      deleteFromApi: function (id) {
        var self = this
        deleteRequest( requestUrl + '/front-system/stock/recipient/'+ id, function (response) {
         self.listData({})
        })
      },
//    查看详情
      gotoDetail: function (id){
        window.location.href = '#!/site/instock/AllotNum/'+ id
      }
    },
    ready: function () {
      this.listData(1)
      var self = this
//      获取制单人
      getDataFromSiteApi( requestUrl + '/front-system/account',{},function(response){
        self.creators = response.data.body.list
      })
    },
    methods: {
//    收货单汇总-列表数据渲染
      listData: function (page) {
        this.$http({
          url: requestUrl + '/front-system/stock/recipient',
          method: 'get',
          data: {
            start_time1: this.query.start_time1 || '',
            end_time1: this.query.end_time1 || '',
            start_time2: this.query.start_time2 || '',
            end_time2: this.query.end_time2 || '',
            order_code: this.query.order_code || '',
            create_person: this.query.create_person || '',
            check_status: this.query.check_status || '',
            page: page,
            per_page: 16
          },
          headers: {'X-Overpowered-Token': token}
        }).then(function (response) {
          this.page = response.data.body.pagination
          console.log(this.page)
          this.list = response.data.body.list
          changeStatus(this.list)
        }, function (err) {
          error(err)
        })
      },
//      搜索页面
      search: function () {
        var self = this
        searchRequest(
          requestUrl + '/front-system/stock/recipient',
          {
            order_number: this.orderNumber,
            checked: Number(this.selectedCheck),
            creator_id: this.createPersonId,
            per_page: 16
          },
          function (response) {
            self.list = response.data.body.list
            self.page = response.data.body.pagination
            exchangeData(self.list)
          }
        )
      }
    },
    data: function () {
      return {
        page: {},
        list: [],
        checkUrl:requestUrl +  '/front-system/stock/recipient/',
        creators: [],
        orderNumber: '',
        selectedCheck: '',
        createPersonId: '',
        detailUrl: '/#!/site/instock/AllotNum/',
        gridColumns: {
          order_number: '单号',
          checked: '审核状态',
          creator: '制单人',
          auditor: '审核人',
          date: '收货日期',
          amount: '收货数量'
        },
        query: {
          start_time1: '',
          start_time2: '',
          end_time1: '',
          end_time2: '',
          order_code: '',
          check_status: '',
          create_person: ''
        }
      }
    }
  }
</script>
<style scoped>
  table thead tr th {
    text-align: center
  }

  table thead tr th:first-child {
    text-align: left;
  }

  table tbody tr td:first-child {
    text-align: left;
  }
</style>
