<!--yml
category: 未分类
date: 2024-05-27 14:59:01
-->

# A love letter to Apache Echarts

> 来源：[https://alicegg.tech//2024/02/14/echarts.html](https://alicegg.tech//2024/02/14/echarts.html)

```
const candles = echarts.init(document.getElementById('candles'));
candles.setOption({
  title: { text: 'Apple Inc. Week 6 2024' },
  tooltip: {},
  dataset: {
    source: [
      ['date', 'close', 'open', 'lowest', 'highest', 'volume', 'sma50'],
      ['2024-02-09', 188.85, 188.65, 188.00, 189.99, 45155216.0, 190.48],
      ['2024-02-08', 188.32, 189.38, 187.35, 189.54, 40962047.0, 190.51],
      ['2024-02-07', 189.41, 190.64, 188.61, 191.05, 53438961.0, 190.54],
      ['2024-02-06', 189.30, 186.86, 186.77, 189.31, 43490762.0, 190.55],
      ['2024-02-05', 187.68, 188.15, 185.84, 189.25, 69668812.0, 190.59],
    ],
  },
  xAxis: {
    type: 'time', // automatically parses the dates
  },
  yAxis: [
    // scaled axis for the price
    { name: 'Price', scale: true },
    // hidden axis for the volume
    {
      max: 150000000,
      scale: true,
      axisLabel: { show: false },
      axisLine: { show: false },
      splitLine: { show: false },
    },
  ],
  series: [
    // this creates a candlestick chart using cols [0-5]
    {
      type: 'candlestick',
      yAxisIndex: 0,
      tooltip: {
	formatter: (param) => `
	  Date: ${param.value[0]}<br />
	  Open: ${param.value[2]}<br />
	  High: ${param.value[4]}<br />
	  Low: ${param.value[3]}<br />
	  Close: ${param.value[1]}<br />
	`,
      },
    },
    // the volume gets mapped to a bar chart
    {
      type: 'bar',
      encode: { x: 'date', y: 'volume' },
      yAxisIndex: 1,
      tooltip: {
	formatter: (param) => `Volume: ${param.value[5]}`,
      },
    },
    // SMA line
    {
      type: 'line',
      encode: { x: 'date', y: 'sma50' },
      yAxisIndex: 0,
      tooltip: {
	formatter: (param) => `SMA50: ${param.value[6]}`,
      },
    },
  ],
}); 
```