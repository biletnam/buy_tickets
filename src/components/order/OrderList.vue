<template>
  <div class="col-sm-9 col-right page">
    <a>
      <strong>
        <i class="glyphicon glyphicon-barcode"></i>
        订单中心
        <i class="glyphicon glyphicon-chevron-right"></i>
        订单列表         
      </strong>
    </a>
    <hr>

    <div class="row flex-box">
      <!--检票状态-->
      <div class="controls btn-group col-md-2  col-condensed flex-item">
        <label>检票状态</label>
        <select v-model="checkStatus" class="form-control">
          <option value="all">全部检票状态</option>
          <option value="checked">已检票</option>
          <option value="checking">检票中</option>
          <option value="waiting">待检票</option>
          <option value="refund">已退票</option>
        </select>
      </div>
      <!--是否过期-->
      <div class="controls btn-group col-md-2  col-condensed flex-item">
        <label>检票状态</label>
        <select v-model="isExpired" class="form-control">
          <option value="all">已过期及未过期</option>
          <option value="yes">已过期</option>
          <option value="no">未过期</option>
        </select>
      </div>
    <!-- </div>
    <hr class="break-space">
    <div class="row"> -->
      <!--成交时间范围-->
      <div class="col-md-2  col-condensed flex-item">
        <label>成交时间范围</label>
        <input id="orderCreateDateRange" type="text" name="daterange" class="form-control overflow-ellipsis" />
      </div>
      <!--游玩时间范围-->
      <div class="col-md-2  col-condensed flex-item">
        <label>游玩时间范围</label>
        <input id="visitDateRange" type="text" name="daterange" class="form-control overflow-ellipsis" />
      </div>
      <!--检票时间范围-->
      <div class="col-md-2  col-condensed flex-item">
        <label>检票时间范围</label>
        <input id="checkDateRange" type="text" name="daterange" class="form-control overflow-ellipsis" />
      </div>
    </div>

    <hr class="break-space">

    <div class="input-group flex-box multi-input">
      <!--预订人姓名-->
      <input class="flex-item form-control" v-model="bookPerson" placeholder="预订人姓名">
      <!--预订人手机号-->
      <input class="flex-item form-control" v-model="bookMobile" placeholder="预订人手机号">
      <!--辅助码-->
      <input class="flex-item form-control" v-model="orderTicketCode" placeholder="辅助码">
      <!--分销商订单号-->
      <input class="flex-item form-control" v-model="parterOrderId" placeholder="分销商订单号">
      <!--景区订单号-->
      <input class="flex-item form-control" v-model="orderId" placeholder="景区订单号">
      <!--商品名-->
      <input class="flex-item form-control" v-model="goodsName" placeholder="商品名">
      <!--证件类型-->
      <select class="flex-item form-control" v-model="bookerIDType" style="border-right:none; max-width:120px;">
        <option value="ID_CARD">身份证</option>
        <option value="ERTONG">儿童无证件</option>
        <option value="GANGAO">港澳通行证</option>
        <option value="HUZHAO">护照</option>
        <option value="SHIBING">士兵证</option>
        <option value="JUNGUAN">军官证</option>
        <option value="HUKOUBO">户口薄</option>
        <option value="CHUSHENGZHENGMING">出生证明</option>
        <option value="TAIBAO">台湾通行证</option>
        <option value="TAIBAOZHENG">台胞证</option>
        <option value="OTHER">其他</option>
      </select>
      <!--证件号-->
      <input class="flex-item form-control" v-model="bookerID" placeholder="证件号">
    </div>
    <hr class="break-space">

    <!--搜索按钮-->
    <div class="controls pull-right">
      <a class="btn btn-primary" @click="this.search">
        <i class="glyphicon glyphicon-search"></i> 搜索
      </a>
    </div>

    <hr class="break-space-double">
    <hr class="break-space-double">
    <hr class="break-space-double">

    
    <!--订单列表-->
    <v-server-table ref="table" url="/otauser" :columns="columns" :options="options">
    </v-server-table>
  </div>
</template>

<script>
import $ from '@/utils/jquery-v'
import Vue from 'vue'
import {ServerTable} from '@/../static/vue-tables-2'
import moment from 'moment'
import {getCardType, getIsExpired, getCheckStatus} from '@/utils/filters'
Vue.use(ServerTable)
Vue.component('odl-tourist', {
  template: '<a class="btn btn-sm btn-primary" href="javascript:void(0);" @click="showTourist">游客</a>',
  methods: {
    showTourist () {
      this.$bus.$emit('touristInit', {
        orderId: this.data.orderId
      })
    }
  },
  props: ['data']
})
Vue.component('odl-refund', {
  template: '<button :disabled="disabled" class="btn btn-sm btn-danger" href="javascript:void(0);" @click="showRefund">退票</button>',
  methods: {
    showRefund () {
      this.$bus.$emit('refundInit', {
        orderId: this.data.orderId,
        waitingCheck: this.checkStatus === 'refund' ? 0 : (this.data.totalTickets - this.data.checkedTickets)
      })
    }
  },
  computed: {
    disabled () {
      return (this.data.totalTickets - this.data.checkedTickets) === 0 || this.data.checkStatus === 'refund'
    }
  },
  props: ['data']
})
let self
let initStartCreateTime = new Date(moment(new Date()).format('YYYY-MM-DD') + ' 00:00:00').getTime()
let initEndCreateTime = initStartCreateTime + 30 * 24 * 60 * 60 * 1000
export default {
  name: 'order-list',
  created () {
    self = this
    $(function () {
      // orderCreateDateRange
      $('input[id="orderCreateDateRange"]').daterangepicker({
        locale: {
          format: 'YYYY-MM-DD',
          applyLabel: '确定',
          cancelLabel: '清空',
          customRangeLabel: 'Custom',
          daysOfWeek: [
            '日',
            '一',
            '二',
            '三',
            '四',
            '五',
            '六'
          ],
          monthNames: [
            '一月',
            '二月',
            '三月',
            '四月',
            '五月',
            '六月',
            '七月',
            '八月',
            '九月',
            '十月',
            '十一月',
            '十二月'
          ]
        },
        startDate: moment(initStartCreateTime).format('YYYY-MM-DD'),
        endDate: moment(initEndCreateTime).format('YYYY-MM-DD')
      },
      function (start, end, label) {
        self.orderCreateDateStart = new Date(start._d).getTime()
        self.orderCreateDateEnd = new Date(end._d).getTime()
        // alert('A new date range was chosen: ' + start.format('YYYY-MM-DD') + ' to ' + end.format('YYYY-MM-DD'))
      })
      $('input[id="orderCreateDateRange"]').on('cancel.daterangepicker', function (ev, picker) {
        $(this).val('')
        self.orderCreateDateStart = ''
        self.orderCreateDateEnd = ''
      })

      // visitDateRange
      $('input[id="visitDateRange"]').daterangepicker({
        autoUpdateInput: false,
        locale: {
          format: 'YYYY-MM-DD',
          applyLabel: '确定',
          cancelLabel: '清空',
          customRangeLabel: 'Custom',
          daysOfWeek: [
            '日',
            '一',
            '二',
            '三',
            '四',
            '五',
            '六'
          ],
          monthNames: [
            '一月',
            '二月',
            '三月',
            '四月',
            '五月',
            '六月',
            '七月',
            '八月',
            '九月',
            '十月',
            '十一月',
            '十二月'
          ]
        }
      })
      $('input[id="visitDateRange"]').on('cancel.daterangepicker', function (ev, picker) {
        $(this).val('')
        self.visitDateStart = ''
        self.visitDateEnd = ''
      })
      $('input[id="visitDateRange"]').on('apply.daterangepicker', function (ev, picker) {
        $(this).val(picker.startDate.format('YYYY-MM-DD') + ' - ' + picker.endDate.format('YYYY-MM-DD'))
        self.visitDateStart = new Date(picker.startDate._d).getTime()
        self.visitDateEnd = new Date(picker.endDate._d).getTime()
      })

      // checkDateRange
      $('input[id="checkDateRange"]').daterangepicker({
        autoUpdateInput: false,
        locale: {
          format: 'YYYY-MM-DD',
          applyLabel: '确定',
          cancelLabel: '清空',
          customRangeLabel: 'Custom',
          daysOfWeek: [
            '日',
            '一',
            '二',
            '三',
            '四',
            '五',
            '六'
          ],
          monthNames: [
            '一月',
            '二月',
            '三月',
            '四月',
            '五月',
            '六月',
            '七月',
            '八月',
            '九月',
            '十月',
            '十一月',
            '十二月'
          ]
        }
      })
      $('input[id="checkDateRange"]').on('cancel.daterangepicker', function (ev, picker) {
        $(this).val('')
        self.checkDateStart = ''
        self.checkDateEnd = ''
      })
      $('input[id="checkDateRange"]').on('apply.daterangepicker', function (ev, picker) {
        $(this).val(picker.startDate.format('YYYY-MM-DD') + ' - ' + picker.endDate.format('YYYY-MM-DD'))
        self.checkDateStart = new Date(picker.startDate._d).getTime()
        self.checkDateEnd = new Date(picker.endDate._d).getTime()
      })
    })
  },
  mounted () {
    // $('input').val(2)
  },
  data () {
    return {
      checkStatus: 'all',
      isExpired: 'all',
      bookMobile: '',
      bookPerson: '',
      orderCreateDateStart: initStartCreateTime,
      orderCreateDateEnd: initEndCreateTime,
      visitDateStart: '',
      visitDateEnd: '',
      checkDateStart: '',
      checkDateEnd: '',
      orderTicketCode: '',
      parterOrderId: '',
      orderId: '',
      goodsName: '',
      bookerIDType: 'ID_CARD',
      bookerID: '',
      columns: ['orderId', 'parterOrderId', 'userOrderId', 'partnerName', 'goodsName', 'price', 'bookPerson', 'bookMobile', 'bookerIDTypeText', 'bookerID', 'orderCreateTime', 'visitDateStart', 'visitDateEnd', 'totalTickets', 'checkedTickets', 'returnedTickets', 'isExpired', 'checkStatusText', 'touristBtn', 'refundBtn'],
      options: {
        perPageValues: [],
        texts: {
          count: '第{from}至{to}条数据／共{count}条数据|共{count}条数据|共1条数据',
          filter: 'Filter Results:',
          filterPlaceholder: 'Search query',
          limit: 'Records:',
          noResults: '查无结果',
          page: 'Page:', // for dropdown pagination
          filterBy: 'Filter by {column}', // Placeholder for search fields when filtering by column
          loading: 'Loading...', // First request to server
          defaultOption: 'Select {column}' // default option for list filters
        },
        sortIcon: {
          up: '',
          down: ''
        },
        filterable: false,
        perPage: 15,
        templates: {
          touristBtn: 'odl-tourist',
          refundBtn: 'odl-refund'
        },
        params: {
          action: 'GetOTAOrderListByPage',
          sortBy: 'OrderCreateTime',
          orderBy: 'desc',
          count: 15, // 一页显示数量
          search: {
            orderId: '',
            parterOrderId: '',
            bookPerson: '',
            bookMobile: '',
            bookerIDType: '',
            bookerID: '',
            goodsName: '',
            checkStatus: 'all',
            partnerName: '',
            orderCreateDateStart: initStartCreateTime,
            orderCreateDateEnd: initEndCreateTime,
            visitDateStart: '',
            visitDateEnd: '',
            checkDateStart: '',
            checkDateEnd: '',
            isExpired: 'all',
            orderTicketCode: ''
          }
        },
        responseAdapter: function (resp) {
          let data = resp.data.orders.lists
          data.map(function (item, index, input) {
            item.orderCreateTime = moment(new Date(item.orderCreateTime - 0)).format('YYYY-MM-DD HH:mm:ss')
            item.visitDateStart = moment(new Date(item.visitDateStart - 0)).format('YYYY-MM-DD HH:mm:ss')
            item.visitDateEnd = moment(new Date(item.visitDateEnd - 0)).format('YYYY-MM-DD HH:mm:ss')
            item.bookerIDTypeText = getCardType(item.bookerIDType)
            item.isExpired = getIsExpired(item.isExpired)
            item.checkStatusText = getCheckStatus(item.checkStatus)
            return item
          })
          return {
            data: data,
            count: resp.data.orders.totalCount
          }
        },
        headings: {
          orderId: '订单号',
          partnerName: '分销商',
          orderCreateTime: '下单时间',
          returnedTickets: '已退',
          isExpired: '过期',
          bookMobile: '手机号',
          visitDateStart: '游玩开始日期',
          bookPerson: '预订人',
          parterOrderId: '分销商订单号',
          totalTickets: '数量',
          price: '单价',
          bookerID: '证件号',
          checkStatusText: '状态',
          checkedTickets: '已检',
          visitDateEnd: '游玩结束日期',
          goodsName: '票名',
          userOrderId: '用户订单号',
          bookerIDTypeText: '证件类型',
          touristBtn: '游客',
          refundBtn: '退票'
        }
      }
    }
  },
  methods: {
    search () {
      console.log(this.visitDateStart)
      this.$refs.table.customQueries = {
        search: {
          checkStatus: this.checkStatus,
          isExpired: this.isExpired,
          orderCreateDateStart: this.orderCreateDateStart,
          orderCreateDateEnd: this.orderCreateDateEnd,
          visitDateStart: this.visitDateStart,
          visitDateEnd: this.visitDateEnd,
          checkDateStart: this.checkDateStart,
          checkDateEnd: this.checkDateEnd,
          bookMobile: this.bookMobile,
          bookPerson: this.bookPerson,
          orderTicketCode: this.orderTicketCode,
          parterOrderId: this.parterOrderId,
          orderId: this.orderId,
          goodsName: this.goodsName,
          bookerIDType: this.bookerIDType,
          bookerID: this.bookerID
        }
      }
      this.$refs.table.refresh()
    }
  },
  components: {
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
</style>
