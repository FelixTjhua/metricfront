<template>
  <!-- 顶部导航栏 -->
  <navigationBar />
  <div class="container">
    <div class="content">
      <el-scrollbar height="85vh" class="scroll-container">
        <!-- UFC计算部分 -->
        <div class="step-section">
          <div class="section-title">步骤1: UFC计算</div>
          <el-table :data="UFCData" class="table ufc-table">
            <el-table-column prop="name" label="请输入对应个数" />
            <el-table-column label="Simple" prop="simple">
              <template #default="scope">
                <el-input
                    v-if="activeIndex === scope.$index"
                    v-model="scope.row.simple"
                    class="input-field"
                    size="small"
                ></el-input>
                <span v-else>{{ scope.row.simple }}</span>
              </template>
            </el-table-column>

            <el-table-column label="Average" prop="average">
              <template #default="scope">
                <el-input
                    v-if="activeIndex === scope.$index"
                    v-model="scope.row.average"
                    class="input-field"
                    size="small"
                ></el-input>
                <span v-else>{{ scope.row.average }}</span>
              </template>
            </el-table-column>

            <el-table-column label="Complex" prop="complex">
              <template #default="scope">
                <el-input
                    v-if="activeIndex === scope.$index"
                    v-model="scope.row.complex"
                    class="input-field"
                    size="small"
                ></el-input>
                <span v-else>{{ scope.row.complex }}</span>
              </template>
            </el-table-column>

            <el-table-column fixed="right" label="操作" class="operations-column">
              <template #default="scope">
                <div v-if="activeIndex === scope.$index">
                  <el-button type="primary" @click="handleSave" size="small">保存</el-button>
                </div>
                <div v-else>
                  <el-button type="success" @click="handleEdit(scope.$index)" size="small">编辑</el-button>
                </div>
              </template>
            </el-table-column>
          </el-table>
          <div class="calculator-result center">
            UFC计算结果: {{ ufcResult }}
          </div>
        </div>

        <!-- VAF计算部分 -->
        <div class="step-section">
          <div class="section-title">步骤2: VAF计算</div>
          <el-table :data="VAFData" class="table vaf-table">
            <el-table-column prop="index" label="序号" />
            <el-table-column prop="character" label="系统特性" />
            <el-table-column prop="description" label="描述" />
            <el-table-column label="等级" prop="level">
              <template #default="scope">
                <el-input
                    v-if="activeIndex === scope.$index"
                    v-model="scope.row.level"
                    class="input-field"
                    size="small"
                ></el-input>
                <span v-else>{{ scope.row.level }}</span>
              </template>
            </el-table-column>

            <el-table-column fixed="right" label="操作" class="operations-column">
              <template #default="scope">
                <div v-if="activeIndex === scope.$index">
                  <el-button type="primary" @click="handleSave" size="small">保存</el-button>
                </div>
                <div v-else>
                  <el-button type="success" @click="handleEdit(scope.$index)" size="small">编辑</el-button>
                </div>
              </template>
            </el-table-column>
          </el-table>
          <div class="calculator-result center">
            VAF计算结果: {{ vafResult }}
          </div>
        </div>

        <!-- 最终计算结果 -->
        <div class="final-result center">
          功能点度量结果:
          <span class="result-value">
            {{ Math.ceil(vafResult * ufcResult * 100) / 100.0 }}
          </span>
        </div>
      </el-scrollbar>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent } from "vue";
import navigationBar from "@/components/navigationBar.vue";

export default defineComponent({
  name: "functionPoint",
  data() {
    return {
      UFCData: [
        { name: "External Inputs", simple: 0, average: 2, complex: 0 },
        { name: "External Outputs", simple: 0, average: 2, complex: 1 },
        { name: "External Queries", simple: 0, average: 2, complex: 0 },
        { name: "Internal Logic Files", simple: 0, average: 0, complex: 1 },
        { name: "External Interfaces Files", simple: 0, average: 2, complex: 0 },
      ],
      VAFData: [
        { index: 0, character: "数据通讯", description: "信息交互", level: 3 },
        { index: 1, character: "数据分布处理", description: "数据分布处理", level: 3 },
        { index: 2, character: "系统性能", description: "系统响应时间", level: 0 },
        { index: 3, character: "使用配置", description: "硬件平台使用情况", level: 5 },
        { index: 4, character: "事务处理频率", description: "事务处理频率", level: 0 },
        { index: 5, character: "在线数据输入", description: "在线数据输入百分比", level: 3 },
        { index: 6, character: "操作便利", description: "操作便捷性", level: 3 },
        { index: 7, character: "在线更新", description: "ILF在线更新", level: 3 },
        { index: 8, character: "处理复杂度", description: "是否复杂处理", level: 0 },
        { index: 9, character: "复用性", description: "系统复用情况", level: 5 },
        { index: 10, character: "安装难易", description: "安装和维护难易", level: 0 },
        { index: 11, character: "运行维护", description: "启动和恢复难易", level: 0 },
        { index: 12, character: "多站点支持", description: "是否多站点支持", level: 0 },
        { index: 13, character: "变更支持", description: "变更支持情况", level: 3 },
      ],
      activeIndex: -1,
    };
  },
  computed: {
    ufcResult() {
      let res = 0;
      const fp_ufc_data = this.UFCData;
      for (let i = 0; i < fp_ufc_data.length; i++) {
        res += fp_ufc_data[i].simple * (i === 0 ? 3 : 4);
        res += fp_ufc_data[i].average * (i === 1 ? 5 : 4);
        res += fp_ufc_data[i].complex * (i === 3 ? 10 : 6);
      }
      return res;
    },
    vafResult() {
      let res = 0;
      const fp_vaf_data = this.VAFData;
      for (let i = 0; i < fp_vaf_data.length; i++) {
        res += fp_vaf_data[i].level * 1;
      }
      return res * 0.01 + 0.65;
    },
  },
  methods: {
    handleEdit(index: number) {
      this.activeIndex = index;
    },
    handleSave() {
      this.activeIndex = -1;
    },
  },
  components: { navigationBar },
});
</script>

<style lang="scss" scoped>
@import "../assets/style/css/functionPoint";
</style>
