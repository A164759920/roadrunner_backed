<template>
  <div class="container">
    <div class="infoTitle"> IP INFORMATION.</div>
    <div class="ipInfo">
      <div id="ipCharts" style="height:400px"> </div>
      <el-table border stripe height="400" :data="state.ipData" :default-sort="{ prop: 'count', order: 'descending' }">
        <el-table-column prop="ipv4" label="ip地址"></el-table-column>
        <el-table-column prop="province" label="省份"></el-table-column>
        <el-table-column prop="city" label="城市"></el-table-column>
        <el-table-column prop="count" label="访问次数" sortable></el-table-column>
      </el-table>
    </div>
    <div class="infoTitle"> USER-AGENT INFORMATION.</div>
    <div class="uaInfo">
      <div id="uaCharts" style="height:400px"> </div>
      <el-table border stripe height="400" :data="state.uaData"
        :default-sort="{ prop: 'lastdate', order: 'descending' }">
        <el-table-column prop="ipv4" label="ip地址"></el-table-column>
        <el-table-column prop="os" label="系统"></el-table-column>
        <el-table-column prop="browser" label="浏览器"></el-table-column>
        <el-table-column prop="device" label="设备"></el-table-column>
        <el-table-column prop="lastdate" label="访问时间"></el-table-column>
      </el-table>

    </div>

  </div>
</template>

<script setup>
// reactive复杂数据类型 ref 基础数据类型
import * as echarts from "echarts";
import { onBeforeMount, onMounted, onUnmounted, reactive } from "vue";
import axios from "axios"

let state = reactive({
  ipData: [],
  uaData: []
})
let IPchartData = []  //IP饼图数据
let UAchartData = []  //UA饼图数据
function getIpData() {
  axios.get("https://server.roadrunner2002.top:8899/back_ip?type=all").then(res => {
    if (res.data.code === 0) {
      state.ipData = res.data.data
    }
  })
  axios.get("https://server.roadrunner2002.top:8899/back_ip?type=top5").then(res => {
    if (res.data.code === 0) {
      // 边界长度处理
      const length = res.data.data.length > 5 ? 5 : res.data.data.length
      for (let i = 0; i < length; i++) {
        const dataObj = {
          value: res.data.data[i].count,
          name: res.data.data[i].province
        }
        IPchartData.push(dataObj)
      }
      initIPChart()
    }
  })
}
function getUAData() {
  // 后端接口支持分页，前端暂时不支持
  axios.get("https://server.roadrunner2002.top:8899/back_ua?type=all&limit=100&offset=1").then(res => {
    if (res.data.code === 0) {
      state.uaData = res.data.data
    }
  })
  axios.get("https://server.roadrunner2002.top:8899/back_ua?type=top5").then(res => {
    if (res.data.code === 0) {
      const length = res.data.data.length > 5 ? 5 : res.data.data.length
      for (let i = 0; i < length; i++) {
        const dataObj = {
          value: res.data.data[i].count,
          name: res.data.data[i].os
        }
        UAchartData.push(dataObj)
      }
      initIUAhart()
    }
  })
}
onBeforeMount(() => {
  getUAData(),
    getIpData()
}
)
onMounted(() => {
})
onUnmounted(() => {
  echarts.dispose
})

function initIPChart() {
  let chart = echarts.init(document.getElementById("ipCharts"))
  chart.setOption({
    backgroundColor: '#545c64de',
    title: {
      text: 'IP归属地 Top5',
      left: 'center',
      top: 20,
      textStyle: {
        color: '#ccc'
      }
    },
    tooltip: {
      trigger: 'item'
    },
    visualMap: {
      show: false,
      min: 0,
      max: IPchartData[0].value,
      inRange: {
        color: ['#e8e8e8', 'yellow', '#ffd04b'],
        symbolSize: [30, 100]

      }
    },
    series: [
      {
        name: 'Province',
        type: 'pie',
        radius: '55%',
        center: ['50%', '50%'],
        data: IPchartData,
        roseType: 'radius',
        label: {
          color: 'rgba(255, 255, 255, 0.3)'
        },
        labelLine: {
          lineStyle: {
            color: 'rgba(255, 255, 255, 0.3)'
          },
          smooth: 0.2,
          length: 10,
          length2: 20
        },
        itemStyle: {
          color: '#c23531',
          shadowBlur: 200,
          shadowColor: 'rgba(0, 0, 0, 0.5)'
        },
        animationType: 'scale',
        animationEasing: 'elasticOut',
      }
    ]
  })
  return chart
}
function initIUAhart() {
  let chart = echarts.init(document.getElementById("uaCharts"))
  chart.setOption({
    backgroundColor: '#545c64de',
    title: {
      text: '访客设备 Top5',
      left: 'center',
      top: 20,
      textStyle: {
        color: '#ccc'
      }
    },
    tooltip: {
      trigger: 'item'
    },
    visualMap: {
      show: false,
      min: 0,
      max: UAchartData[0].value,
      inRange: {
        color: ['#e8e8e8', 'yellow', '#ffd04b'],
        symbolSize: [30, 100]

      }
    },
    series: [
      {
        name: 'Device',
        type: 'pie',
        radius: '55%',
        center: ['50%', '50%'],
        data: UAchartData,
        roseType: 'radius',
        label: {
          color: 'rgba(255, 255, 255, 0.3)'
        },
        labelLine: {
          lineStyle: {
            color: 'rgba(255, 255, 255, 0.3)'
          },
          smooth: 0.2,
          length: 10,
          length2: 20
        },
        itemStyle: {
          color: '#c23531',
          shadowBlur: 200,
          shadowColor: 'rgba(0, 0, 0, 0.5)'
        },
        animationType: 'scale',
        animationEasing: 'elasticOut',
      }
    ]
  })
  return chart
}
</script>

<style>
.container {
  width: 100%;
  height: 500px;
}

.el-table{
  font-family: Russo One, Arial, sans-serif;
  font-weight: 600;
  font-size: 20px;
}

.infoTitle {
  width: 100%;
  height: 60px;
  background-color: #545c64;
  color: whitesmoke;
  font-family: Russo One, Arial, sans-serif;
  font-weight: 600;
  font-size: 20px;
  text-align: center;
  line-height: 60px;
  border-bottom: 5px solid  rgb(255 208 75);
}

.container .ipInfo {
  width: 100%;
  display: flex;
}

.ipInfo #ipCharts {
  width: 50%;
}

.container .uaInfo {
  width: 100%;
  display: flex;
}

.uaInfo #uaCharts {
  width: 50%;
}
</style>
