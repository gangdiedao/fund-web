<template>
  <div class="app-container">
    <el-row style="margin-bottom: 15px;" type="flex" class="row-bg" justify="space-between">
      <el-col :span="12">
      </el-col>
      <el-col :span="12" style="text-align: right;">
        <el-input placeholder="股票名称/股票代码" v-model.trim="query.keyword" clearable @clear="search" size="mini" style="width: 240px;">
          <el-button slot="append" icon="el-icon-search" @click="search">搜索</el-button>
        </el-input>
      </el-col>
    </el-row>
    <el-table
      v-loading="loading"
      :data="tableData">
      <el-table-column
        prop="name"
        label="股票名称">
      </el-table-column>
      <el-table-column
        prop="code"
        label="股票代码">
        <template slot-scope="{row}">
          <el-link type="primary" :href="`https://emwap.eastmoney.com/quota/hq/stock?id=${row.code}&mk=${row.code[0] == '6' ? 1 : 0}`" target="_blank">{{row.code}}</el-link>
        </template>
      </el-table-column>
      <el-table-column
        prop="count"
        label="机构数量">
      </el-table-column>
      <el-table-column
        prop="updatedAt"
        label="更新时间">
        <template slot-scope="{row}">
          <span>{{ row.updatedAt | parseTime('{y}-{m}-{d} {h}:{i}:{s}') }}</span>
        </template>
      </el-table-column>
      <el-table-column
        label="操作"
        width="140px">
        <template slot-scope="{row}">
          <el-button type="primary" size="mini" @click="handleShowDetail(row)">持仓机构</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-row type="flex" justify="end">
      <pagination v-show="total>0" :total="total" :page.sync="query.page" :limit.sync="query.limit" @pagination="getList" />
    </el-row>
    <el-dialog
      title="持仓机构"
      :visible.sync="dialogVisible"
      width="60%">
      <el-table
        :data="item"
        show-summary
        :summary-method="getSummaries">
        <el-table-column
          label="基金名称"
          prop="name">
        </el-table-column>
        <el-table-column
          label="基金代码"
          prop="code">
          <template slot-scope="{row}">
            <el-link type="primary" :href="`https://m.1234567.com.cn/index.html?page=jjxq&code=${row.code}`" target="_blank">{{row.code}}</el-link>
          </template>
        </el-table-column>
        <el-table-column
          label="持有市值"
          prop="markval">
          <template slot-scope="{row}">
          <span>{{ row.markval | parsePrice }}</span>
        </template>
        </el-table-column>
      </el-table>
    </el-dialog>
  </div>
</template>

<script>
import Pagination from '@/components/Pagination'

export default {
  components: {
    Pagination 
  },
  filters: {
    parsePrice(val) {
      return (val/100000000).toFixed(3) + '亿'
    }
  },
  data() {
    return {
      code: '',
      query: {
        page: 1,
        limit: 10,
        keyword: ''
      },
      total: 0,
      tableData: [],
      loading: false,
      showInput: false,

      dialogVisible: false,
      item: []
    }
  },
  created() {
    this.getList()
  },
  methods: {
    getSummaries(param) {
      const { columns, data } = param;
      console.log(data)
      const sums = [];
      columns.forEach((column, index) => {
        if (index === 0) {
          sums[index] = '合计';
          return;
        }
        if (index === 1) {
          sums[index] = '';
          return;
        }
        if (index === 2) {
          const values = data.map(item => Number(item.markval))
          let count = values.reduce((a, b) => {
            return a + b
          }, 0)
          sums[index] = (count / 100000000).toFixed(3) + '亿'
        }
      });
      return sums;
    },
    handleShowDetail(item) {
      this.item = item.lists
      this.dialogVisible = true
    },
    search() {
      this.query.page = 1
      this.getList()
    },
    getList() {
      this.loading = true
      this.$request({
        url: '/shares',
        method: 'get',
        params: this.query
      }).then(res => {
        this.total = res.total
        this.tableData = res.data
      }).finally(() => {
        this.loading = false
      })
    }
  }
}
</script>

<style>

</style>