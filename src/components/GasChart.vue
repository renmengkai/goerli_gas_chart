<script setup>
import * as echarts from "echarts";
import {onBeforeUnmount, onMounted, ref} from "vue";
import * as ethers from "ethers";

const data = ref([])
const timer = ref(null);

onMounted(() => {
  let dom = document.getElementById('chart-container');
  let myChart = echarts.init(dom, null, {
    renderer: 'canvas',
    useDirtyRect: false
  });
  const provider = new ethers.JsonRpcProvider('https://opbnb-mainnet-rpc.bnbchain.org', 204)
  provider.getFeeData().then((feeData) => {
    data.value.push([
      new Date().toLocaleTimeString().replace(/^\D*/, ''),
      ethers.formatUnits(feeData.gasPrice, 'gwei')
    ]);
    freshChart(myChart)
  });
  timer.value = setInterval(() => {
    if (data.value.length > 20) {
      data.value.shift();
    }
    provider.getFeeData().then((feeData) => {
      data.value.push([
        new Date().toLocaleTimeString().replace(/^\D*/, ''),
        ethers.formatUnits(feeData.gasPrice, 'gwei')
      ]);
      freshChart(myChart)
    });
  }, 1000 * 3);

  window.addEventListener('resize', myChart.resize);
});

const freshChart = (myChart) => {
  const dateList = data.value.map(function (item) {
    return item[0];
  });
  const valueList = data.value.map(function (item) {
    return item[1];
  });
  const option = {
    title: [
      {
        left: 'center',
        text: 'opBNB mainNet gas price （ Gwei ）',
      }
    ],
    xAxis: {
      type: 'category',
      data: dateList
    },
    yAxis: {
      type: 'value'
    },
    series: [
      {
        data: valueList,
        type: 'line',
        smooth: true,
        areaStyle: {}
      }
    ]
  };
  myChart.setOption(option);
}

onBeforeUnmount(() => {
  clearInterval(timer.value);
  window.removeEventListener('resize', myChart.resize);
});
</script>

<template>
  <div id="chart-container">
  </div>
</template>

<style scoped>
#chart-container {
  width: 100%;
  min-width: 1200px;
  height: 100%;
  min-height: 600px;
}
</style>
