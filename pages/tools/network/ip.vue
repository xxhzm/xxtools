<template>
  <div class="container">
    <div class="nav-header">
      <el-button link @click="router.push('/')" class="back-btn">
        <el-icon><ArrowLeft /></el-icon>
        返回首页
      </el-button>
    </div>

    <div class="tool-header">
      <el-icon class="header-icon"><Location /></el-icon>
      <div class="header-text">
        <h1>IP查询</h1>
        <p class="subtitle">查询IP地址的地理位置、运营商等详细信息</p>
      </div>
    </div>

    <div class="tool-content">
      <div class="input-section">
        <el-input
          v-model="ip"
          placeholder="请输入IP地址，如: 1.1.1.1"
          :prefix-icon="Monitor"
          clearable
          @keyup.enter="queryIP"
        >
          <template #append>
            <el-button @click="queryIP" :loading="loading"> 查询 </el-button>
          </template>
        </el-input>
      </div>

      <div v-if="result" class="result-section">
        <div class="result-card">
          <div class="result-item">
            <span class="label">IP地址：</span>
            <span class="value">{{ result.ip }}</span>
          </div>
          <div class="result-item">
            <span class="label">地理位置：</span>
            <span class="value">{{ result.address }}</span>
          </div>
          <div class="result-item">
            <span class="label">运营商：</span>
            <span class="value">{{
              result.isp === "0" ? "获取失败" : result.isp || "未知"
            }}</span>
          </div>
          <div class="result-item">
            <span class="label">经纬度：</span>
            <span class="value">{{ result.lat }}, {{ result.lng }}</span>
          </div>
        </div>
      </div>
    </div>

    <div class="usage-guide">
      <h2>使用说明</h2>
      <ol>
        <li>输入需要查询的IP地址</li>
        <li>点击"查询"按钮或按回车键</li>
        <li>查看IP地址的详细信息</li>
        <li>如果有经纬度信息，可以在地图上查看位置</li>
      </ol>
      <div class="note">
        <el-icon><InfoFilled /></el-icon>
        <span>提示：支持IPv4和IPv6地址查询，不输入IP则查询当前IP</span>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from "vue";
import { ElMessage } from "element-plus";
import { useRouter } from "vue-router";
import {
  Location,
  Monitor,
  InfoFilled,
  ArrowLeft,
} from "@element-plus/icons-vue";
import toolsData from "~/data/data.json";

const router = useRouter();
const ip = ref("");
const loading = ref(false);
const result = ref<any>(null);

// 从data.json导入工具信息
const toolInfo = toolsData.tools.find((tool) => tool.id === 14);

// 更新页面标题和描述
const title = toolInfo?.title || "IP查询";
const description =
  toolInfo?.description || "查询IP地址的地理位置、运营商等详细信息";

useHead({
  title: `${title} - ${toolsData.subtitle}`,
  meta: [
    {
      name: "description",
      content: description,
    },
    {
      name: "keywords",
      content: toolInfo?.keywords.join(",") || "",
    },
  ],
});

const queryIP = async () => {
  loading.value = true;
  result.value = null;

  try {
    const response = await fetch(
      `https://v2.xxapi.cn/api/ip?ip=${encodeURIComponent(ip.value)}`
    );
    const data = await response.json();

    if (data.code === 200 && data.data) {
      result.value = data.data;
      ElMessage.success("查询成功");
    } else {
      ElMessage.error(data.msg || "查询失败，请检查IP地址是否正确");
    }
  } catch (error) {
    ElMessage.error("网络错误，请稍后重试");
  } finally {
    loading.value = false;
  }
};

// 初始查询当前IP
queryIP();
</script>

<style scoped>
.container {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
}

.nav-header {
  margin-bottom: 20px;
}

.back-btn {
  display: flex;
  align-items: center;
  gap: 4px;
  font-size: 14px;
  color: var(--el-text-color-regular);
  transition: color 0.3s;
}

.back-btn:hover {
  color: var(--el-color-primary);
}

.tool-header {
  display: flex;
  align-items: center;
  gap: 16px;
  margin-bottom: 32px;
}

.header-icon {
  font-size: 32px;
  color: var(--el-color-primary);
  background: var(--el-color-primary-light-9);
  padding: 12px;
  border-radius: 12px;
}

.header-text h1 {
  font-size: 24px;
  margin-bottom: 8px;
  color: var(--el-text-color-primary);
}

.subtitle {
  color: var(--el-text-color-secondary);
  font-size: 14px;
}

.tool-content {
  background: var(--el-bg-color);
  border-radius: 8px;
  padding: 24px;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.05);
  margin-bottom: 24px;
}

.input-section {
  margin-bottom: 24px;
}

.result-section {
  padding-top: 24px;
  border-top: 1px solid var(--el-border-color-light);
}

.result-card {
  background: var(--el-bg-color-page);
  border-radius: 8px;
  padding: 20px;
  border: 1px solid var(--el-border-color);
  margin-bottom: 20px;
}

.result-item {
  display: flex;
  margin-bottom: 16px;
  line-height: 1.6;
}

.result-item:last-child {
  margin-bottom: 0;
}

.result-item .label {
  width: 80px;
  color: var(--el-text-color-secondary);
  flex-shrink: 0;
}

.result-item .value {
  color: var(--el-text-color-primary);
  word-break: break-all;
}

.usage-guide {
  background: var(--el-bg-color);
  border-radius: 8px;
  padding: 24px;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.05);
}

.usage-guide h2 {
  font-size: 18px;
  margin-bottom: 16px;
  color: var(--el-text-color-primary);
}

.usage-guide ol {
  margin: 0;
  padding-left: 20px;
  color: var(--el-text-color-regular);
}

.usage-guide li {
  margin-bottom: 8px;
  line-height: 1.6;
}

.note {
  display: flex;
  align-items: center;
  gap: 8px;
  margin-top: 16px;
  padding: 12px;
  background: var(--el-color-primary-light-9);
  border-radius: 6px;
  color: var(--el-text-color-secondary);
  font-size: 14px;
}

.note .el-icon {
  color: var(--el-color-primary);
}

@media (max-width: 768px) {
  .container {
    padding: 15px;
  }
}
</style>
