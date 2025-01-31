<template>
  <div class="container">
    <div class="nav-header">
      <el-button link @click="router.push('/')" class="back-btn">
        <el-icon><ArrowLeft /></el-icon>
        返回首页
      </el-button>
    </div>

    <div class="tool-header">
      <el-icon class="header-icon"><Search /></el-icon>
      <div class="header-text">
        <h1>Whois查询</h1>
        <p class="subtitle">查询域名注册信息、到期时间、DNS等详细信息</p>
      </div>
    </div>

    <div class="tool-content">
      <div class="input-section">
        <el-input
          v-model="domain"
          placeholder="请输入域名，如: xxhzm.cn"
          :prefix-icon="Monitor"
          clearable
          class="domain-input"
        />
        <el-button
          type="primary"
          class="query-btn"
          @click="startQuery"
          :loading="loading"
        >
          开始查询
        </el-button>
      </div>

      <div v-if="result" class="result-section">
        <div class="result-card">
          <div class="result-item">
            <span class="label">域名：</span>
            <span class="value">{{
              result.data.data.domain_name || "无数据"
            }}</span>
          </div>
          <div class="result-item">
            <span class="label">注册商：</span>
            <span class="value">{{
              result.data.data.sponsoring_registrar || "无数据"
            }}</span>
          </div>
          <div class="result-item">
            <span class="label">注册人：</span>
            <span class="value">{{
              result.data.data.registrant || "无数据"
            }}</span>
          </div>
          <div class="result-item">
            <span class="label">注册时间：</span>
            <span class="value">{{
              result.data.data.registration_time || "无数据"
            }}</span>
          </div>
          <div class="result-item">
            <span class="label">到期时间：</span>
            <span class="value">{{
              result.data.data.expiration_time || "无数据"
            }}</span>
          </div>
          <div class="result-item">
            <span class="label">域名状态：</span>
            <span class="value">{{
              result.data.data.domain_status || "无数据"
            }}</span>
          </div>
          <div class="result-item">
            <span class="label">DNS服务器：</span>
            <span class="value">{{
              Array.isArray(result.data.data.dns_serve)
                ? result.data.data.dns_serve.join(", ")
                : "无数据"
            }}</span>
          </div>
        </div>
      </div>
    </div>

    <div class="usage-guide">
      <h2>使用说明</h2>
      <ol>
        <li>在输入框中输入需要查询的域名</li>
        <li>点击"开始查询"按钮</li>
        <li>等待查询完成，查看域名的详细信息</li>
      </ol>
      <div class="note">
        <el-icon><InfoFilled /></el-icon>
        <span
          >注意：查询结果仅供参考，具体信息请以域名注册商提供的信息为准</span
        >
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from "vue";
import { ElMessage } from "element-plus";
import { useRouter } from "vue-router";
import {
  Monitor,
  Search,
  InfoFilled,
  ArrowLeft,
} from "@element-plus/icons-vue";
import toolsData from "~/data/data.json";

const router = useRouter();
const domain = ref("");
const result = ref<any>(null);
const loading = ref(false);

// 从data.json导入工具信息
const toolInfo = toolsData.tools.find((tool) => tool.id === 4);

// 更新页面标题和描述
const title = toolInfo?.title || "Whois查询";
const description =
  toolInfo?.description || "查询域名注册信息、到期时间、DNS等详细信息";

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
    {
      name: "viewport",
      content:
        "width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=0",
    },
  ],
});

const startQuery = async () => {
  if (!domain.value.trim()) {
    ElMessage.warning("请输入需要查询的域名");
    return;
  }

  loading.value = true;
  try {
    const response = await fetch(
      `https://v2.xxapi.cn/api/whois?domain=${encodeURIComponent(domain.value)}`
    );
    const data = await response.json();

    if (data.code === 200 && data.data) {
      result.value = data;
    } else {
      ElMessage.error(data.msg || "查询失败，请稍后重试");
    }
  } catch (error) {
    ElMessage.error("网络错误，请稍后重试");
  } finally {
    loading.value = false;
  }
};
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
  display: flex;
  gap: 16px;
  flex-wrap: wrap;
}

.domain-input {
  flex: 1;
  min-width: 200px;
}

.query-btn {
  margin-top: 16px;
  width: 100%;
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
  width: 100px;
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
  .input-section {
    flex-direction: column;
  }
}
</style>
