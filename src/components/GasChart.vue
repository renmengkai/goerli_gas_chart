<script setup>
import * as echarts from "echarts";
import {onBeforeUnmount, onMounted, reactive, ref} from "vue";
import * as ethers from "ethers";

const data = ref([])
const timer = ref(null);

onMounted(() => {
  let dom = document.getElementById('chart-container');
  let myChart = echarts.init(dom, null, {
    renderer: 'canvas',
    useDirtyRect: false
  });
  const provider = new ethers.JsonRpcProvider('https://ethereum-goerli.publicnode.com')

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
    visualMap: [
      {
        show: false,
        type: 'continuous',
        seriesIndex: 0,
        min: 0,
        max: 100
      }
    ],
    title: [
      {
        left: 'center',
        text: 'Goerli Testnet gas price （ gwei ）',
      }
    ],
    tooltip: {
      trigger: 'axis'
    },
    xAxis: [
      {
        data: dateList
      },
    ],
    yAxis: [
      {},
    ],
    grid: [
      {
        bottom: '10%'
      },
      {
        top: '10%'
      }
    ],
    series: [
      {
        type: 'line',
        showSymbol: false,
        data: valueList
      },
    ]
  }
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
