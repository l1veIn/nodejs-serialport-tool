<style>
* {
  margin: 0;
  padding: 0;
  overflow: unset;
}

html,
body,
.el-container,
.el-aside {
  height: 100%;
}

html,
body {
  background-color: #f8f8f8;
}

.el-container {
  padding: 10px;
}

.el-aside,
.el-main,
.el-footer {
  padding: 5px;
}

.el-main,
.el-footer {
  height: auto;
}

.el-aside {
  margin-right: 5px;
  overflow: unset;
}

/* .el-card__body{
  padding: 0
} */

.el-textarea {
  border-bottom: 1px solid #ebeef5;
}

textarea {
  border: none;
}

.el-textarea__inner {
  border: none;
}

.buttonGroup {
  padding-top: 10px;
  display: flex;
  justify-content: flex-end;
}

button {
  width: 100px;
  /* height: 30px; */
  margin-left: 10px;
}

.el-switch {
  color: lightgrey;
  margin-bottom: 10px;
}

.el-switch__label {
  font-size: 15px;
}

.el-card__body {
  padding: 10px;
}

.el-form-item__label,
.el-tabs__item {
  font-weight: 600;
}

.el-form {
  margin-bottom: 10px;
}

.el-form-item {
  margin-bottom: 0;
}
</style>

<template>
<el-container>
  <el-aside style="" width="200px">
    <el-card style="height:100%">
      <el-form :label-position="labelPosition" label-width="80px" :model="formLabelAlign">
        <el-form-item label="选择接口">
          <el-select v-model="selectedValue" placeholder="请选择">
            <el-option v-for="(item,index) in ports" :key="index" :label="item.comName" :value="index">
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="波特率">
          <el-input v-model="baudRate"></el-input>
        </el-form-item>
        <el-form-item label="最多显示">
          <el-input v-model="maxLength"></el-input>
        </el-form-item>
      </el-form>
      <el-switch v-model="userMySQL" active-text="mySQL">
      </el-switch>
      <transition name="el-fade-in-linear">
        <!-- <div v-show="userMySQL" class="transition-box">.el-fade-in-linear</div> -->
        <el-form v-show="userMySQL" :label-position="labelPosition" label-width="80px" :model="formLabelAlign">
          <el-form-item label="用户名">
            <el-input v-model="username"></el-input>
          </el-form-item>
          <el-form-item label="密码">
            <el-input v-model="passwd"></el-input>
          </el-form-item>
        </el-form>
      </transition>
      <el-button :type="started?'warning':'success'" style="width:100%" round @click='started?stopWatching():startWatching()'>{{ started? '停止':'开始'}}</el-button>
    </el-card>
  </el-aside>
  <el-container style="padding:0">
    <el-main>
      <el-card style="height:100%">
        <el-tabs v-model="activeName" @tab-click="handleClick" stretch>
          <el-tab-pane label="数据" name="first">
              <el-table
    :data="source"
    height='330'
    stripe
    style="width: 100%">
    <el-table-column
      prop=2
      label="时间"
      width="250">
    </el-table-column>
    <el-table-column
      prop=1
      label="电平值/V"
      width="250">
    </el-table-column>
  </el-table>
          </el-tab-pane>
          <el-tab-pane label="图像" name="second">
            <el-switch v-model="animation" active-text="animation">
            </el-switch>
            <div id="myChart" :style="{width: '595px', height: '300px'}"></div>
          </el-tab-pane>
        </el-tabs>
      </el-card>
    </el-main>
     <div style="height:14px;width:100%;font-size:14px;color:lightgrey;text-align:right;margin-right:20px">已开源:https://github.com/l1veIn/nodejs-serialport-tool</div>
    <el-footer style="height: 154px">
      <el-card style="height:100%">
        <!-- <textarea name="name" rows="8" cols="80"></textarea> -->
        <el-input type="textarea" :rows="3" placeholder="请输入内容" v-model="textarea">
        </el-input>
        <div class="buttonGroup">
          <el-button type="info" @click='clearText'>清空</el-button>
          <el-button type="primary" @click='sendMsg'>发送</el-button>
        </div>
      </el-card>
    </el-footer>
   
  </el-container>

</el-container>
</template>

<script>
import Serialport from "serialport";
import echarts from "echarts";
import { setInterval } from "timers";
export default {
  name: "test",
  data() {
    return {
      text: "",
      labelPosition: "top",
      formLabelAlign: {
        name: "",
        region: "",
        type: ""
      },
      selectedValue: 0,
      ports: [],
      port: null,
      baudRate: "9600",
      activeName: "first",
      textarea: "",
      userMySQL: false,
      source: [],
      maxLengthNum: 100,
      animation: true,
      int: null,
      fakeDataClock: null,
      started: false
    };
  },
  computed: {
    option() {
      return {
        legend: {},
        tooltip: {},
        dataset: {
          dimensions: [
            {
              name: "time",
              type: "time"
            },
            // 可以简写为 string，表示维度名。
            "value"
          ],
          // 获取实时数据
          source: this.source
        },
        xAxis: {
          type: "time"
        },
        yAxis: {
          type: "value",
          name: "电平值/V",
          nameTextStyle: {
            fontSize: 20
          },
          splitLine: {
            show: false
          },
          axisLabel: {
            formatter: "{value} V"
          }
        },
        series: [
          {
            type: "line",
            encode: {
              // 将 "time" 列映射到 X 轴。
              x: "time",
              // 将 "value" 列映射到 Y 轴。
              y: "value",
              tooltip: ["time", "value"]
            }
          }
        ],
        animation: this.animation
      };
    },
    maxLength: {
      get: function() {
        return this.maxLengthNum; //获取的时候直接获取值
      },
      set: function(value) {
        if (Object.is(value * 1, NaN)) {
          this.$notify({
            title: "警告",
            message: "请输入数字",
            type: "warning",
            offset: 10
          });
        } else {
          this.maxLengthNum = value;
        }
      }
    }
  },
  mounted() {
    console.log("app mounted");
    Serialport.list((err, ports) => {
      console.log(ports);
      this.ports = ports;
      if (ports.length == 0) {
        this.$message({
          showClose: true,
          message: "没有检测到可用串口",
          type: "warning"
        });
      }
    });
  },
  methods: {
    drawLine() {
      let that = this;
      console.log("drawLine");
      // 基于准备好的dom，初始化echarts实例
      let myChart = echarts.init(document.getElementById("myChart"));
      this.int = self.setInterval(function() {
        //   console.log("drawLine");
        //   console.log(myChart)
        let temp = that.source[0];
        // 测试输入
        //执行绘图
        myChart.setOption(that.option);
      }, 500);
    },
    stopDrawLine() {
      clearInterval(this.int);
      this.int = null;
    },
    handleClick(e) {
      let that = this;
      console.log(e.$options.propsData);
      // if (e.$options.propsData.name == "second") {
      //   that.drawLine()
      // } else {
      //   if (this.int) {
      //     that.stopDrawLine()
      //   }
      // }
    },
    clearText() {
      this.textarea = "";
    },
    sendMsg() {
      if (this.textarea == "") {
        this.$notify({
          title: "警告",
          message: "发送内容不能为空",
          type: "warning",
          offset: 10
        });
      } else {
        // 发送要发送的内容
        this.$notify({
          title: "成功",
          message: "发送成功",
          type: "success",
          offset: 10
        });
      }
    },
    startWatching() {
      console.log("startWatching");
      let that = this;
      that.source = []
      that.getData();
    },
    stopWatching() {
      console.log("stopWatching");
      let that = this;
      that.stopDrawLine();
      that.stopGetData();
      that.started = false;
    },
    getData() {
      let that = this;
      if (!that.ports[that.selectedValue]) {
        that.$message({
          showClose: true,
          message: "没有检测到可用串口",
          type: "warning"
        });
        return;
      }
      that.port = new Serialport(
        that.ports[that.selectedValue].comName,
        {
          baudRate: that.baudRate * 1
        },
        function(err) {
          if (err) {
            return that.$notify({
              title: "警告",
              message: err.message,
              type: "warning",
              offset: 10
            });
            // console.log('Error: ', err.message);
          } else {
            that.$notify({
              title: "成功",
              message: "连接成功，正在读取数据。。",
              type: "success",
              offset: 10,
              duration: 1000
            });
            that.port.on("data", data => {
              console.log("Received:\t", new Date().toLocaleTimeString()+':'+data[0]+'V');
              let hexData = data.map(function(x){
                  return parseInt(x).toString(16)
              })
              let  hexNum = hexData.join('')
              let realNum = parseInt(hexNum,16)
              that.source.push([
                Date.now(),realNum,new Date().toLocaleTimeString()
                ]);
              that.source.length > that.maxLength &&
                that.source.splice(0, that.source.length - that.maxLength);
            });
            // 监听数据
            // that.port.on('data', (data) => {
            //   console.log('Received:\t', data.toString('utf8'));
            // });
            // 开始绘图
            that.drawLine();
            that.started = true;
          }
        }
      );
    },
    stopGetData() {
      // clearInterval(this.fakeDataClock);
      // this.fakeDataClock = null;
      let that = this
      that.port.close()
      that.port = null;
      
    }
  },
  components: {
    // covButton
  }
};
</script>
