<template>
  <admin-nav></admin-nav>
  <div class="container-fluid">
    <div class="row">
      <left-production></left-production>
      <div class="col-lg-10">
        <!-- 路径导航 -->
        <ol class="breadcrumb">
          <li class="active"><span class="glyphicon glyphicon-home c-erp" aria-hidden="true"></span> 您当前的位置：生产首页</li>
          <li class="active">工厂生产</li>
          <li class="active">查看工厂生产详情</li>
        </ol>
        <!--详情页面-->
        <summary-detail
          :table-header="gridColumns"
          :table-data="list"
          :grid-operate="gridOperate"
          :check-url="checkUrl"
          :edit-flag.sync="editFlag"
          :is-exist="isExist"
          :has-validate-authority="authority.validate"
          :has-edit-authority="authority.edit"
        >
        </summary-detail>
        <!--有列表切换的时候的情况-->
        <ul class="nav nav-tabs" role="tablist">
          <li role="presentation" class="active" @click="changeActive($event)" id="1"><a href="javascript:void(0)"
                                                                                         data-toggle="tab">入库明细</a></li>
          <li role="presentation" @click="changeActive($event)" id="2"><a href="javascript:void(0)" data-toggle="tab">入库汇总</a>
          </li>
          <a :href="exports" target="_blank" v-if="authority.exports"><span class="btn btn-info spanblocks fr">导出</span></a>
        </ul>
        <!-- Tab panes -->
        <div class="tab-content">
          <!-- 入库明细 -->
          <div role="tabpanel" class="tab-pane active" v-if="detailModal">
            <!--表格详情列表-->
            <table class="table table-striped table-bordered table-hover">
              <thead>
              <tr class="text-center">
                <th v-for="value in  gridColumns2">
                  {{value}}
                </th>
              </tr>
              </thead>
              <tbody>
              <tr class="text-center" v-for="entry in detailList" track-by="$index" :id="[entry.id ? entry.id : '']">
                <td>{{entry.item_code}}</td>
                <td>{{entry.item_name}}</td>
                <td>{{entry.unit_specification}}</td>
                <td>{{entry.origin_stock_amount}}{{entry.unit_name}}</td>
                <td>{{entry.demand_amount}}{{entry.unit_name}}</td>
                <td v-if='editFlag'>
                  <count :count.sync='entry.main_reference_value'></count>
                  {{entry.unit_name}}
                </td>
                <td v-if='!editFlag'>{{entry.main_reference_value}}{{entry.unit_name}}</td>
                <td>{{entry.reference_number}}</td>
              </tr>
              </tbody>
            </table>
          </div>

          <!-- 入库汇总 -->
          <div role="tabpanel" class="tab-pane active" v-if="summaryModal">
            <!--表格详情列表-->
            <table class="table table-striped table-bordered table-hover">
              <thead>
              <tr class="text-center">
                <th v-for="value in  gridColumns2">
                  {{value}}
                </th>
              </tr>
              </thead>
              <tbody>
              <tr class="text-center" v-for="entry in summarystockGoods" track-by="$index"
                  :id="[entry.id ? entry.id : '']">
                <td>{{entry.item_code}}</td>
                <td>{{entry.item_name}}</td>
                <td>{{entry.unit_specification}}</td>
                <td>{{entry.origin_stock_amount}}{{entry.unit_name}}</td>
                <td>{{entry.demand_amount}}{{entry.unit_name}}</td>
                <td>{{entry.item_amount}}{{entry.unit_name}}</td>
                <td>{{entry.reference_number}}</td>
              </tr>
              </tbody>
            </table>
          </div>
        </div>
      </div>
    </div>
  </div>
  <!--错误信息-->
  <error-tip :err-modal.sync="modal.errModal" :err-info="modal.errInfo"></error-tip>
</template>
<style>
</style>
<script>
  import $ from 'jquery'
  import Grid from '../../../common/Grid'
  import Page from '../../../common/Page'
  import AdminNav from '../../AdminNav'
  import Modal from '../../../common/Modal'
  import Summary from '../../../common/Summary'
  import DatePicker from  '../../../common/DatePicker'
  import LeftProduction from '../../common/LeftProduction'
  import SummaryDetail from '../../../common/SummaryDetail'
  import Count from '../../../common/Count'
  import ErrorTip from '../../../common/ErrorTip'
  import {
    requestUrl,
    putDataToApi,
    requestSystemUrl,
    getDataFromApi,
    token,
    exchangeData,
    detailNull,
    searchRequest,
    deleteRequest,
    checkRequest,
    systermAuthority,
    finishRequest
  } from '../../../../publicFunction/index'
  export default{
    components: {
      Grid: Grid,
      Page: Page,
      Modal: Modal,
      AdminNav: AdminNav,
      Summary: Summary,
      DatePicker: DatePicker,
      LeftProduction: LeftProduction,
      SummaryDetail: SummaryDetail,
      Count: Count,
      ErrorTip: ErrorTip
    },
    events: {
//    绑定翻页事件
      pagechange: function (currentpage) {
        var self = this
        this.$http({
          url: requestSystemUrl + '/backend-system/purchase/purchase',
          data: {
            page: currentpage
          },
          method: 'get',
          headers: {'X-Overpowered-Token': token}
        }).then(function (response) {
          self.page = response.data.body.pagination
          self.list = response.data.body.list
          exchangeData(this.list)
        }, function (err) {
          console.log(err)
        })
      },
//      编辑
      editGoods: function (event) {
        this.editFlag = true
        this.isExist = true
      },
//      删除请求
      deleteFromApi: function (id) {
        var self = this
        deleteRequest(requestSystemUrl + '/backend-system/produce/factory/' + id, function (response) {
          console.log('deleted')
        })
      },
//     完成請求
      finishFromApi: function (id) {
        var self = this
        finishRequest(requestSystemUrl + '/backend-system/produce/factory/' + id + '/finished', function (response) {
          console.log('finished')
        })
      },
//      保存
      saveGoods: function (event) {
        var self = this
        var id = this.$route.params.queryId
        var item = []
        $.each(self.detailList, function (index, val) {
          var obj = {}
          obj['reference_id'] = val.item_id
          obj['id'] = val.id
          obj['amount'] = val.main_reference_value
          obj['reference_type'] = val.item_type
          item.push(obj)
        })
        var data = {
          items: item
        }
        var url = requestSystemUrl + '/backend-system/produce/factory/' + id
        putDataToApi(url, data, function (res) {
          self.isExist = false
          self.editFlag = false
          self.listData()
        }, function (err) {
          self.isExist = true
          self.editFlag = true
          self.modal.errModal = true
          self.modal.errInfo = '服务器错误'
        })
      }
    },
    ready: function () {
      this.listData()
//    权限判断
//      审核
      if(systermAuthority.indexOf('factory-produce-list-check') > -1){
        this.authority.validate = true
      }
//      编辑
      if(systermAuthority.indexOf('factory-produce-list-edit') > -1){
        this.authority.edit = true
      }
//      导出
      if(systermAuthority.indexOf('factory-produce-list-export') > -1){
        this.authority.exports = true
      }
    },
    methods: {
      listData: function (page) {
        var currentId = this.$route.params.queryId
        var self = this
//        获取商品列表详情
        var url = requestSystemUrl + '/backend-system/produce/factory/' + currentId
//       获取单个列表详情
        var purchaseUrl = requestSystemUrl + '/backend-system/produce/factory/' + currentId + '/get'
//       获取商品列表详情
        getDataFromApi(url, {}, function (response) {
          self.detailList = response.data.body.list
        })
//        获取采购列表详情
        getDataFromApi(purchaseUrl, {}, function (response) {
          self.list = response.data.body
          exchangeData(self.list)
        })
      },
//      切换
      changeActive: function (event) {
        var cur = $(event.currentTarget)
        cur.addClass('active').siblings('li').removeClass('active')
        switch (Number(cur.attr('id'))) {
          case 1:
            this.detailModal = true
            this.summaryModal = false
            break
          case 2:
            this.detailModal = false
            this.summaryModal = true
            this.summary()
        }
      },
      //          汇总方法
      summary: function () {
        var self = this
        self.summaryPrice = 0
        this.summarystockGoods = []
        this.summarystockGoods = this.summarystockGoods.concat(self.detailList)
        $.each(this.summarystockGoods, function (index, val) {
          val.item_amount = val.main_reference_value
        })
        this.summarystockGoods = this.summaryMethod("item_code", this.summarystockGoods)
      },
//     汇总方法
      summaryMethod: function (ObjPropInArr, array) {
        var hash = {};
        var result = [];
        for (var i = 0; i < array.length; i++) {
          if (hash[array[i][ObjPropInArr]]) {
            hash[array[i][ObjPropInArr]].item_amount = Number(array[i].item_amount) + Number(hash[array[i][ObjPropInArr]].item_amount)
            hash[array[i][ObjPropInArr]].demand_amount = Number(array[i].demand_amount) + Number(hash[array[i][ObjPropInArr]].demand_amount)
            hash[array[i][ObjPropInArr]].origin_stock_amount = Number(array[i].origin_stock_amount) + Number(hash[array[i][ObjPropInArr]].origin_stock_amount)
          } else {
            hash[array[i][ObjPropInArr]] = array[i];
          }
        }
        for (var j in hash) {
          result.push(hash[j])
        }
        return result
      }
    },
    computed: {
//      导出
      exports: function () {
        var url = requestSystemUrl + '/backend-system/' + token + '/export' + '/produce/factory'
        return this.exportUrl = url + '/export-excel/' + this.$route.params.queryId
      }
    },
    data: function () {
      return {
        page: [],
        list: {},
        summarystockGoods: [],
        checkUrl: requestSystemUrl + '/backend-system/produce/factory/',
        detailList: [],
        editFlag: false,
        isExist: false,
        detailModal: true,
        summaryModal: false,
        gridOperate: true,
        modal: {
          errModal: false,
          errInfo: ''
        },
        gridColumns: {
          document_number: '生产单号',
          checked: '审核状态',
          creator_name: '制单人',
          auditor_name: '审核人',
          created_at: '制单日期',
          operated_at: '生产日期',
          amount: '生产数量'
        },
        gridColumns2: {
          item_code: "货号",
          item_name: "品名",
          unit_specification: '单位规格',
          origin_stock_amount: '总部库存',
          demand_amount: '门店要货数量',
          main_reference_value: '生产数量',
          reference_number: '来源要货单号',
        },
        authority: {
          validate: false,
          edit: false,
          exports: false
        }
      }
    }
  }
</script>
