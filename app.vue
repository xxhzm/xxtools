<template>
  <el-config-provider :locale="zhCn">
    <NuxtPage></NuxtPage>
  </el-config-provider>
  <el-backtop :right="100" :bottom="100" />
</template>

<script setup>
import zhCn from "element-plus/dist/locale/zh-cn.mjs";
// 添加全局配置，设置中文
const locale = ref(zhCn);
</script>
