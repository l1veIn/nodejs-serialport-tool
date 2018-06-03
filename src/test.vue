<template>

  <el-container>
  <el-aside style="" width="200px">
  <el-form :label-position="labelPosition" label-width="80px" :model="formLabelAlign">
  <el-form-item label="选择接口">
    <el-input v-model="formLabelAlign.name"></el-input>
  </el-form-item>
  <el-form-item label="波特率">
    <el-input v-model="formLabelAlign.region"></el-input>
  </el-form-item>
</el-form>

  <el-button type="success" round>start</el-button>
</el-aside>
  <el-main>  <div id="myChart" :style="{width: '300px', height: '300px'}"></div></el-main>
</el-container>


</template>

<script>
import Serialport from 'serialport'
import echarts from 'echarts'
// import covButton from './button'
export default {
  name: 'test',
  data() {
    return {
      text: '',
	     labelPosition: 'top',
        formLabelAlign: {
          name: '',
          region: '',
          type: ''
        }
    }
  },
  computed: {
    ports() {
      Serialport.list((err, ports) => {
        console.log(ports);
      });
      return ports
    }
  },
  mounted() {
    this.drawLine();
  },
  methods: {
    drawLine() {
      // 基于准备好的dom，初始化echarts实例
      let myChart = echarts.init(document.getElementById('myChart'))
      // 绘制图表
      myChart.setOption({
        title: {
          text: '在Vue中使用echarts'
        },
        tooltip: {},
        xAxis: {
          data: ["衬衫", "羊毛衫", "雪纺衫", "裤子", "高跟鞋", "袜子"]
        },
        yAxis: {},
        series: [{
          name: '销量',
          type: 'line',
          data: [5, 20, 36, 10, 10, 20]
        }]
      });
    }
  },
  components: {
  // covButton
}
}
</script>

<style>
*{
  margin:0;
  padding: 0
}
html,body,.el-container,.el-aside,.el-main{
  height: 100%;
}
html,body{
  background-color: #F8F8F8;
}
.el-container{
  padding: 10px
}
.el-aside,.el-main{
  background-color: white;
}
.el-aside{
  margin-right: 10px;
  padding: 10px
}

</style>
