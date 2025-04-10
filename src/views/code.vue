<template>
  <!-- 顶部导航栏 -->
  <navigationBar />
  <div class="container">
    <div class="header">
      <div class="header_name">
        <!-- {{ page_name }} -->
      </div>
      <div class="upload">
        <!-- 上传按钮 -->
        <button class="upload-btn" @click="triggerFileInput">
          <input
              class="upload-content"
              ref="fileInput"
              multiple
              type="file"
              webkitdirectory
              @change="handleFileChange"
              style="display: none;"
          />
          <span class="upload-label">上传文件</span>
        </button>
      </div>
    </div>

    <div class="content">
      <el-scrollbar height="80vh">
        <div id="ck-graph" class="ck-graph" v-show="graph"></div>

        <div class="table-name">CK度量</div>
        <el-table :data="CKData" class="table" @row-click="openDetails">
          <el-table-column prop="className" label="ClassName" />
          <el-table-column prop="type" label="Type" />
          <el-table-column prop="wmc" label="WMC" />
          <el-table-column prop="rfc" label="RFC" />
          <el-table-column prop="dit" label="DIT" />
          <el-table-column prop="noc" label="NOC" />
          <el-table-column prop="cbo" label="CBO" />
          <el-table-column prop="lcom" label="LCOM" />
        </el-table>

        <div class="table-name">LK度量</div>
        <el-table :data="LKData" class="table">
          <el-table-column prop="className" label="ClassName" />
          <el-table-column prop="type" label="Type" />
          <el-table-column prop="cs" label="CS" />
          <el-table-column prop="noo" label="NOO" />
          <el-table-column prop="noa" label="NOA" />
          <el-table-column prop="si" label="SI" />
        </el-table>

        <div class="table-name">传统度量</div>
        <el-table :data="TRAData" class="table">
          <el-table-column prop="className" label="ClassName" />
          <el-table-column prop="type" label="Type" />
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
  name: "code",
  data() {
    return {
      CKData: [],  // CK度量数据
      LKData: [],  // LK度量数据
      TRAData: [], // 传统度量数据
      uuid: "",    // 用于上传后获取的UUID
      graph: false, // 控制图表显示
      files: [],   // 存储选择的文件
    };
  },
  components: { navigationBar },
  methods: {
    // 触发文件选择框显示
    triggerFileInput() {
      const fileInput: HTMLInputElement | null = this.$refs.fileInput as HTMLInputElement;
      if (fileInput) {
        fileInput.click();
      }
    },
    // 处理文件选择
    handleFileChange(event: Event) {
      const target = event.target as HTMLInputElement;
      this.files = target.files || [];
      this.uploadFiles(); // 调用上传文件的方法
    },
    // 上传文件
    uploadFiles() {
      if (!this.files || this.files.length === 0) {
        console.error("No files selected");
        return;
      }

      let formData = new FormData();
      for (let i = 0; i < this.files.length; i++) {
        // 只上传java文件
        if (this.files[i].name.endsWith(".java")) {
          formData.append("files", this.files[i]);
        }
      }

      if (formData.entries().next().done) {
        window.alert("请选择Java项目！");
      } else {
        let config = {
          method: "post",
          url: "/api/uploadFiles",  // 你实际的文件上传接口
          data: formData,
        };
        axios(config)
            .then((response) => {
              this.uuid = response.data.data;
              this.getData(); // 上传成功后调用获取数据的方法
            })
            .catch((error) => {
              console.log("Error uploading files:", error);
            });
      }
    },
    // 获取数据
    getData() {
      let that = this;
      metric(this.uuid)
          .then((res: any) => {
            that.CKData = res.data.ck;
            that.LKData = res.data.lk;
            that.TRAData = res.data.tradition;
            that.ckGraph(); // 获取数据后绘制图表
          })
          .catch((err: any) => {
            console.log("Error getting data:", err);
          });
    },
    // 绘制CK度量图表
    ckGraph() {
      let chartDom = document.getElementById("ck-graph");
      let myChart = echarts.init(chartDom);
      let option = {
        title: {
          text: "CK度量图表",
        },
        radar: {
          indicator: [
            { name: "WMC", max: 10 },
            { name: "RFC", max: 30 },
            { name: "DIT", max: 5 },
            { name: "NOC", max: 10 },
            { name: "CBO", max: 20 },
            { name: "LCOM", max: 10 },
          ],
        },
        series: [
          {
            name: "CK度量",
            type: "radar",
            data: this.formatCKGraphData(),
          },
        ],
      };
      myChart.setOption(option);
    },
    // 格式化CK度量图表数据
    formatCKGraphData() {
      let formattedData = [];
      for (const row of this.CKData) {
        formattedData.push({
          value: [
            row.wmc,
            row.rfc,
            row.dit,
            row.noc,
            row.cbo,
            row.lcom,
          ],
          name: row.className,
        });
      }
      return formattedData;
    },
    // 点击表格行查看详细信息
    openDetails(row) {
      let ckGraphData = [];
      let ckName = [];
      let temp = { value: [], name: row.className };
      temp.value.push(row.wmc, row.rfc, row.dit, row.noc, row.cbo, row.lcom);
      ckGraphData.push(temp);
      ckName.push(row.className);
      this.ckGraph(); // 绘制图表
    },
  },
});
</script>

<style lang="scss" scoped>
@import "../assets/style/css/code";
</style>
