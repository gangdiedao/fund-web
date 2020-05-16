<template>
  <div class="app-container">
    <el-row style="margin-bottom: 15px;" type="flex" class="row-bg" justify="space-between">
      <el-col :span="12">
        <el-input v-show="showInput" v-model="code" type="number" size="mini" style="width: 240px" placeholder="请填写基金代码"></el-input>
        <el-button type="primary" size="mini" @click="handleAdd">{{!showInput ? '添加基金' : '确认添加'}}</el-button>
      </el-col>
      <el-col :span="12" style="text-align: right;">
        <el-input placeholder="基金名称/基金代码" v-model.trim="query.keyword" clearable @clear="search" size="mini" style="width: 480px;">
          <el-button slot="append" icon="el-icon-search" @click="search">搜索</el-button>
        </el-input>
      </el-col>
    </el-row>
    <el-table
      v-loading="loading"
      :data="tableData"
      @sort-change="sortChange">
      <el-table-column
        prop="name"
        label="基金名称">
      </el-table-column>
      <el-table-column
        prop="code"
        label="基金代码">
      </el-table-column>
      <el-table-column
        sortable="custom"
        prop="scale"
        label="基金规模">
        <template slot-scope="{row}">
          <span>{{ row.scale | parsePrice }}</span>
        </template>
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
          <el-button type="primary" size="mini" @click="handleShowDetail(row)">持仓明细</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-row type="flex" justify="end">
      <pagination v-show="total>0" :total="total" :page.sync="query.page" :limit.sync="query.limit" @pagination="getList" />
    </el-row>
    <el-dialog
      title="持仓明细"
      :visible.sync="dialogVisible"
      width="60%">
      <el-table
        :data="item">
        <el-table-column
          label="股票名称"
          prop="name">
        </el-table-column>
        <el-table-column
          label="股票代码"
          prop="code">
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
      sort: '',
      dialogVisible: false,
      item: []
    }
  },
  created() {
    this.getList()
  },
  methods: {
    sortChange({ order }) {
      this.query.sort = ''
      if (order === 'ascending') {
        this.query.sort = 'ASC'
      }
      if (order === 'descending') {
        this.query.sort = 'DESC'
      }
      this.search()
    },
    handleShowDetail(item) {
      this.item = item.lists
      this.dialogVisible = true
    },
    search() {
      this.query.page = 1
      this.getList()
    },
    handleAdd() {
      if(this.showInput) {
        if (/\d{6}/.test(this.code)) {
          this.$request({
            url: '/fund/add',
            method: 'post',
            data: {
              code: this.code
            }
          }).then(res => {
            this.$message({
              message: 'success',
              type: 'success'
            })
            this.code = ''
            this.search()
          }).finally(() => {
            this.showInput = false
          })
        }
      } else {
        this.showInput = true
      }
    },
    getList() {
      this.loading = true
      this.$request({
        url: '/fund',
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