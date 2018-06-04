<style>

* {
    margin: 0;
    padding: 0
}

html,
body,
.el-container,
.el-aside {
    height: 100%;
}

html,
body {
    background-color: #F8F8F8;
}

.el-container {
    padding: 10px
}

.el-aside,
.el-main,
.el-footer {
    padding: 5px
}

.el-main,
.el-footer {
    height: auto;
}

.el-aside {
    margin-right: 5px;
}


/* .el-card__body{
  padding: 0
} */

.el-textarea {
    border-bottom: 1px solid #ebeef5
}

.el-footer {
    /* margin-top: 10px; */
}

textarea {
    border: none
}

.el-textarea__inner {
    border: none
}

.buttonGroup {
    padding-top: 10px;
    padding-bottom: 10px;
    display: flex;
    justify-content: flex-end;
}

button {
    width: 120px;
    margin-left: 10px
}

.el-switch {
    color: lightgrey;
    margin-bottom: 10px;
}

.el-switch__label {
    font-size: 15px
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
            <el-button type="success" style="width:100%" round>start</el-button>
        </el-card>
    </el-aside>
    <el-container style="padding:0">
        <el-main>
            <el-card style="height:100%">
                <el-tabs v-model="activeName" @tab-click="handleClick">
                    <el-tab-pane label="数据" name="first">数据</el-tab-pane>
                    <el-tab-pane label="图像" name="second">
                        <div id="myChart" :style="{width: '300px', height: '300px'}"></div>
                    </el-tab-pane>
                </el-tabs>
            </el-card>
        </el-main>
        <el-footer style="height: 200px">
            <el-card style="height:100%">
                <!-- <textarea name="name" rows="8" cols="80"></textarea> -->
                <el-input type="textarea" :rows="5" placeholder="请输入内容" v-model="textarea">
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
            },
            selectedValue: 0,
            ports: [],
            baudRate: '9600',
            activeName: 'first',
            textarea: '',
            userMySQL: false
        }
    },
    computed: {},
    mounted() {
        // this.drawLine();
        Serialport.list((err, ports) => {
            this.ports = ports
        });
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
            },
            handleClick(e) {
                console.log(e.$options.propsData);
                if (e.$options.propsData.name == 'second') {
                    this.drawLine()
                }
            },
            clearText() {
                this.textarea = ''
            },
            sendMsg() {
                if (this.textarea == '') {
                    this.$notify({
                        title: '警告',
                        message: '发送内容不能为空',
                        type: 'warning',
                        offset: 100
                    });
                } else {
                    this.$notify({
                        title: '成功',
                        message: '发送成功',
                        type: 'success',
                        offset: 100
                    });
                }
            }
    },
    components: {
        // covButton
    }
}

</script>
