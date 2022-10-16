---
title: Echarts初体验
date: 2022-10-06 20:29:02
tags: 
- Vue
- Echarts
- 前端
index_img: 
banner_img: /img/article3.webp
categories:
- 前端
comment: waline
---

### 一、Echarts前言

> ECharts 是一个使用 JavaScript 实现的开源可视化库，涵盖各行业图表，满足各种需求。
>
> ECharts 遵循 Apache-2.0 开源协议，免费商用。
>
> ECharts，可以流畅的运行在 PC 和移动设备上，兼容当前绝大部分浏览器（IE9/10/11，Chrome，Firefox，Safari等），底层依赖矢量图形库 [ZRender](https://github.com/ecomfe/zrender)，提供直观，交互丰富，可高度个性化定制的数据可视化图表。

### 二、获取Apache ECharts

> Apache ECharts 支持多种下载方式，我们以从 [jsDelivr](https://www.jsdelivr.com/package/npm/echarts) CDN 上获取为例，介绍如何快速安装。
>
> 在 https://www.jsdelivr.com/package/npm/echarts 选择 `dist/echarts.js`，点击并保存为 `echarts.js` 文件。

### 三、引入 Apache ECharts

> 在`static`目录下保存 `echarts.js` 文件并新建一个 `index.html` 文件，内容如下：

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <!-- 引入刚刚下载的 ECharts 文件 -->
    <script src="echarts.js"></script>
  </head>
</html>
```

### 四、绘制一个图表

> 在绘图前我们需要为 ECharts 准备一个定义了高宽的 DOM 容器。在`index.html` 文件的 `</head>` 之后，添加：

```html
<body>
  <!-- 为 ECharts 准备一个定义了宽高的 DOM -->
  <div id="main" style="width: 600px;height:400px;"></div>
</body>
```

> 然后就可以通过 [echarts.init](https://echarts.apache.org//api.html#echarts.init) 方法初始化一个 echarts 实例并通过 [setOption](https://echarts.apache.org//api.html#echartsInstance.setOption) 方法生成一个简单的柱状图，下面是完整代码。

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <title>ECharts</title>
    <!-- 引入刚刚下载的 ECharts 文件 -->
    <script src="echarts.js"></script>
  </head>
  <body>
    <!-- 为 ECharts 准备一个定义了宽高的 DOM -->
    <div id="main" style="width: 600px;height:400px;"></div>
    <script type="text/javascript">
      // 基于准备好的dom，初始化echarts实例
      var myChart = echarts.init(document.getElementById('main'));

      // 指定图表的配置项和数据
      var option = {
        title: {
          text: 'ECharts 入门示例'
        },
        tooltip: {},
        legend: {
          data: ['销量']
        },
        xAxis: {
          data: ['衬衫', '羊毛衫', '雪纺衫', '裤子', '高跟鞋', '袜子']
        },
        yAxis: {},
        series: [
          {
            name: '销量',
            type: 'bar',
            data: [5, 20, 36, 10, 10, 20]
          }
        ]
      };

      // 使用刚指定的配置项和数据显示图表。
      myChart.setOption(option);
    </script>
  </body>
</html>
```

> vue项目中使用示例完整代码

```html
<template>
  <div class="box">
    <!-- 为 ECharts 准备一个具备大小（宽高）的 DOM -->
    <div ref="chart" style="height:600px;"></div>
  </div>
</template>

<script>
  export default {
    mounted() {
      this.getEchartData();
    },
    methods: {
      getEchartData() {
        let myChart = this.$echarts.init(this.$refs.chart);

        this.$ajax.get("/report/getMemberReport").then(res => {
          myChart.setOption({
            title: {
              text: '会员数量'
            },
            tooltip: {},
            legend: {
              data: ['会员数量']
            },
            xAxis: {
              data: res.data.monthList //动态数据
            },
            yAxis: {
              type: 'value'
            },
            series: [{
              name: '会员数量',
              type: 'line',
              data: res.data.memberCount
            }]
          });
        });
      }
    }
  }
</script>

<style>
</style>

```

## 参考
[^1]: Echarts示例
[^2]: Echarts-Github

<script src="//cdn.jsdelivr.net/npm/@waline/client"></script>
