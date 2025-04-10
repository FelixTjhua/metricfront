<template>
  <el-menu :default-active="activeIndex" class="menu" @select="menuSelect" background-color="transparent" text-color="#ffffff" active-text-color="#00bcd4" mode="vertical">
    <!-- 顶部导航栏 -->
    <div class="menu-header">
      <div class="menu-header-icon">
        <el-icon><Folder /></el-icon>
        <el-icon><Message /></el-icon>
      </div>
      <div class="menu-header-title">
        <h2 class="menu-title">软件度量</h2>
        <p class="menu-group-name">木子石石石</p>
      </div>
    </div>

    <!-- 当前页面的标题 -->
    <h4 class="menu-subtitle">{{ place }}</h4>

    <!-- 菜单项 -->
    <el-menu-item index="0" class="menu-item">
      <el-icon class="menu-item-icon">
        <Reading />
      </el-icon>
      <span class="menu-item-content">功能点度量</span>
    </el-menu-item>

    <el-menu-item index="1" class="menu-item">
      <el-icon class="menu-item-icon">
        <Reading />
      </el-icon>
      <span class="menu-item-content">用例图点度量</span>
    </el-menu-item>

    <el-menu-item index="2" class="menu-item">
      <el-icon class="menu-item-icon">
        <Reading />
      </el-icon>
      <span class="menu-item-content">代码度量</span>
    </el-menu-item>

    <el-menu-item index="3" class="menu-item">
      <el-icon class="menu-item-icon">
        <Reading />
      </el-icon>
      <span class="menu-item-content">AI分析</span>
    </el-menu-item>
  </el-menu>
</template>

<script lang="ts">
import { defineComponent } from "vue";

export default defineComponent({
  name: "NavigationBar",
  data() {
    return {
      activeIndex: "",
      user: {
        username: "yueyue",
        roleId: 1,
        roleName: "系统管理员",
      },
      userType: 1,
      place: "index",
    };
  },
  mounted() {
    this.ready();
  },
  methods: {
    ready() {
      let route = this.$route.path;
      switch (route) {
        case "/":
          this.activeIndex = "0";
          this.place = "functionPoint";
          break;
        case "/useCase":
          this.activeIndex = "1";
          this.place = "useCase";
          break;
        case "/code":
          this.activeIndex = "2";
          this.place = "code";
          break;
        case "/aiAnalysis":
          this.activeIndex = "3";
          this.place = "aiAnalysis";
          break;
        default:
          this.activeIndex = "0";
          this.place = "functionPoint";
          break;
      }
    },
    menuSelect(index: string) {
      switch (index) {
        case "0":
          this.$router.push("/");
          this.activeIndex = "0";
          this.place = "functionPoint";
          break;
        case "1":
          this.$router.push("/useCase");
          this.activeIndex = "1";
          this.place = "useCase";
          break;
        case "2":
          this.$router.push("/code");
          this.activeIndex = "2";
          this.place = "code";
          break;
        case "3":
          this.$router.push("/aiAnalysis");
          this.activeIndex = "3";
          this.place = "aiAnalysis";
          break;
        default:
          this.$router.push("/");
          this.activeIndex = "0";
          this.place = "functionPoint";
          break;
      }
    },
  },
});
</script>

<style lang="scss" scoped>
@import "../assets/style/components/navigationBar";
</style>
