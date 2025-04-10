<template>
  <!-- 顶部导航栏 -->
  <navigationBar />
  <div class="container">
    <!-- 头部部分 -->
    <div class="header">
      <div class="actions">
        <!-- 下载按钮 -->
        <button class="download-btn" @click="downloadResults">
          下载文件
        </button>
      </div>
    </div>

    <!-- 内容部分 -->
    <div class="content">
      <el-scrollbar height="80vh">
        <!-- AI 分析图表 -->
        <div id="ai-graph" class="ai-graph" v-show="graphVisible"></div>

        <!-- AI 分析结果表格 -->
        <div class="table-name">AI 分析结果</div>
        <el-table :data="AIData" class="table" @row-click="openDetails">
          <el-table-column prop="className" label="类名" />
          <el-table-column prop="type" label="类型" />
          <el-table-column prop="accuracy" label="准确率" />
          <el-table-column prop="precision" label="精确度" />
          <el-table-column prop="recall" label="召回率" />
          <el-table-column prop="f1score" label="F1 分数" />
        </el-table>

        <!-- 传统度量表格 -->
        <div class="table-name">传统度量</div>
        <el-table :data="TRAData" class="table">
          <el-table-column prop="className" label="类名" />
          <el-table-column prop="type" label="类型" />
          <el-table-column prop="loc" label="LOC" />
          <el-table-column prop="cp" label="CP" />
          <el-table-column prop="cc" label="CC" />
        </el-table>
      </el-scrollbar>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent } from "vue";
import navigationBar from "@/components/navigationBar.vue";
import { metric } from "@/api/code";
import axios from "axios";
import * as echarts from "echarts";

export default defineComponent({
  name: "aiAnalysis",
  data() {
    return {
      AIData: [], // AI 分析结果数据
      TRAData: [], // 传统度量数据
      uuid: "",
      graphVisible: false, // 控制图表显示
    };
  },
  components: { navigationBar },
  mounted() {},
  methods: {
    getAIData() {
      let that = this;
      metric(this.uuid)
          .then((res: any) => {
            that.AIData = res.data.aiResults; // AI 分析结果
            that.TRAData = res.data.tradition; // 传统度量数据
            that.aiGraph();
          })
          .catch((err: any) => {
            console.log(err);
          });
    },
    aiGraph() {
      let chartDom = document.getElementById("ai-graph");
      let myChart = echarts.init(chartDom);
      let option;
      this.graphVisible = true;

      // AI 分析数据可视化：准确率、精确度、召回率
      option = {
        title: {
          text: "AI 分析图表",
          textStyle: {
            fontSize: 18,
            fontWeight: "bold",
            color: "#333",
          },
        },
        legend: {
          data: ["准确率", "精确度", "召回率", "F1 分数"],
          textStyle: {
            fontSize: 14,
          },
        },
        radar: {
          indicator: [
            { name: "准确率", max: 1 },
            { name: "精确度", max: 1 },
            { name: "召回率", max: 1 },
            { name: "F1 分数", max: 1 },
          ],
          shape: "circle",
          splitArea: {
            areaStyle: {
              color: "rgba(255, 255, 255, 0.2)",
            },
          },
        },
        series: [
          {
            name: "AI 分析",
            type: "radar",
            data: this.formatAIData(),
            symbolSize: 6,
            itemStyle: {
              color: "#67b7dc",
            },
            lineStyle: {
              width: 2,
              color: "#33c6db",
            },
            areaStyle: {
              opacity: 0.2,
            },
          },
        ],
      };
      myChart.setOption(option);
    },
    formatAIData() {
      let formattedData = [];
      for (const result of this.AIData) {
        let temp = {
          value: [result.accuracy, result.precision, result.recall, result.f1score],
          name: result.className,
        };
        formattedData.push(temp);
      }
      return formattedData;
    },
    openDetails(row) {
      // 点击行时查看详细信息
      let aiGraphData = [];
      let temp = { value: [], name: "" };
      temp.name = row.className;
      temp.value.push(row.accuracy);
      temp.value.push(row.precision);
      temp.value.push(row.recall);
      temp.value.push(row.f1score);
      aiGraphData.push(temp);
      this.aiGraph();
    },
    downloadResults() {
      if (!this.AIData.length) {
        alert("没有可下载的数据！");
        return;
      }
      const data = JSON.stringify({
        aiResults: this.AIData,
        tradition: this.TRAData,
      });
      const blob = new Blob([data], { type: "application/json" });
      const link = document.createElement("a");
      link.href = URL.createObjectURL(blob);
      link.download = "AI_分析结果.json";
      link.click();
    },
  },
});
</script>

<style lang="scss" scoped>
@import "../assets/style/css/aiAnalysis"; // 自定义样式
</style>
