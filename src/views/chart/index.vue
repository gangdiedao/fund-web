<template>
  <div class="app-container">
    <el-row style="margin-bottom: 15px;" type="flex" class="row-bg" justify="space-between">
      <el-col :span="24">
        <el-input
          type="textarea"
          :rows="2"
          placeholder="名称"
          style="width: 200px;"
          :disabled="tableData.length"
          v-model="formData.name">
        </el-input>
        <el-input
          type="textarea"
          :rows="2"
          placeholder="排名"
          style="width: 200px;"
          v-model="formData.value">
        </el-input>
        <el-date-picker
          v-model="formData.date"
          type="date"
          value-format="yyyy-MM-dd"
          placeholder="选择日期">
        </el-date-picker>
        <el-button type="primary" size="mini" @click="handleAdd" :loading="btnloading">添加</el-button>
      </el-col>
    </el-row>
    <div id="ecchart" style="width:100%; height:600px;"></div>
  </div>
</template>

<script>
import echarts from 'echarts'
export default {
  data() {
    return {
      query: {
        page: 1,
        limit: 100,
      },
      formData: {
        name: localStorage.getItem('pmb'),
        value: '',
        date: ''
      },
      tableData: [],
      btnloading: false,
      loading: false,
      echarts: null
    }
  },

  mounted() {
    this.$nextTick(() => {
      this.echarts = echarts.init(document.getElementById('ecchart'));
      this.getList()
    })
  },
  methods: {
    search() {
      this.query.page = 1
      this.getList()
    },
    handleAdd() {
      if(this.formData.name && this.formData.value && this.formData.date) {
        if(this.formData.name.split('\n').length != this.formData.value.split('\n').length) {
          return this.$message.error('排名条目对不上')
        }
        localStorage.setItem('pmb', this.formData.name)
        this.btnloading = true
        this.$request({
          url: '/chart',
          method: 'post',
          data: {
            name: this.formData.name.split('\n'),
            datas: this.formData.value.split('\n'),
            dates: this.formData.date
          }
        }).then(res => {
          this.getList()
        }).finally(() => {
          this.btnloading = false
        })
      }
    },
    getList() {
      this.loading = true
      this.$request({
        url: '/chart',
        method: 'get',
        params: this.query
      }).then(res => {
        // this.total = res.total
        this.tableData = res.data
        if(res.data.length) {
          this.setChart(res.data)
        }
      }).finally(() => {
        this.loading = false
      })
    },
    setChart(data) {
      this.echarts.setOption({
          title: {
              text: '走势图',
              subtext: '日赚百万'
          },
          tooltip: {
              trigger: 'axis'
          },
          legend: {
              data: data.map(item => item.name)
          },
          toolbox: {
              show: true,
              feature: {
                  dataZoom: {
                      yAxisIndex: 'none'
                  },
                  dataView: {readOnly: false},
                  magicType: {type: ['line', 'bar']},
                  restore: {},
                  saveAsImage: {}
              }
          },
          xAxis: {
              type: 'category',
              boundaryGap: false,
              data: data[0].dates
          },
          yAxis: {
              type: 'value',
              axisLabel: {
                  formatter: '{value}'
              }
          },
          series: data.map(item => {
            return {
              name: item.name,
              type: 'line',
              data: item.datas
            }
          }),
      })
    }
  }
}
</script>

<style>

</style>
