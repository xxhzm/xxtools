<template>
  <div class="container">
    <div class="nav-header">
      <el-button link @click="router.push('/')" class="back-btn">
        <el-icon><ArrowLeft /></el-icon>
        返回首页
      </el-button>
    </div>

    <div class="tool-header">
      <el-icon class="header-icon"><Warning /></el-icon>
      <div class="header-text">
        <h1>状态码查询</h1>
        <p class="subtitle">检测网站的HTTP状态码和可访问性</p>
      </div>
    </div>

    <div class="tool-content">
      <div class="input-section">
        <el-input
          v-model="url"
          placeholder="请输入网址，如: https://xxapi.cn"
          :prefix-icon="Monitor"
          clearable
          class="url-input"
          @keyup.enter="startCheck"
        />
        <el-button
          type="primary"
          class="check-btn"
          @click="startCheck"
          :loading="loading"
        >
          开始检测
        </el-button>
      </div>

      <div v-if="result" class="result-section">
        <div class="result-card">
          <div class="result-item">
            <span class="label">检测地址：</span>
            <span class="value">{{ testedUrl }}</span>
          </div>
          <div class="result-item">
            <span class="label">状态码：</span>
            <span class="value">
              <el-tag :type="getStatusType(result)">{{ result }}</el-tag>
            </span>
          </div>
          <div class="result-item">
            <span class="label">状态说明：</span>
            <span class="value">{{ getStatusDescription(result) }}</span>
          </div>
        </div>
      </div>
    </div>

    <div class="usage-guide">
      <h2>使用说明</h2>
      <ol>
        <li>在输入框中输入需要检测的网址</li>
        <li>点击"开始检测"按钮或按回车键</li>
        <li>等待检测完成，查看网站的状态码和说明</li>
      </ol>
      <div class="note">
        <el-icon><InfoFilled /></el-icon>
        <span
          >注意：常见的HTTP状态码：200(正常)、301/302(重定向)、404(未找到)、500(服务器错误)</span
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
  Warning,
  InfoFilled,
  ArrowLeft,
} from "@element-plus/icons-vue";
import toolsData from "~/data/data.json";

const router = useRouter();
const url = ref("");
const result = ref("");
const loading = ref(false);
const testedUrl = ref("");

// 从data.json导入工具信息
const toolInfo = toolsData.tools.find((tool) => tool.id === 9);

// 更新页面标题和描述
const title = toolInfo?.title || "状态码查询";
const description = toolInfo?.description || "检测网站的HTTP状态码和可访问性";

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

const startCheck = async () => {
  if (!url.value.trim()) {
    ElMessage.warning("请输入需要检测的网址");
    return;
  }

  loading.value = true;
  try {
    const response = await fetch(
      `https://v2.xxapi.cn/api/status?url=${encodeURIComponent(url.value)}`
    );
    const data = await response.json();

    if (data.code === 200) {
      result.value = data.data;
      testedUrl.value = url.value;
      ElMessage.success("检测完成");
    } else {
      ElMessage.error(data.msg || "检测失败，请稍后重试");
    }
  } catch (error) {
    ElMessage.error("网络错误，请稍后重试");
  } finally {
    loading.value = false;
  }
};

// 获取状态码类型
const getStatusType = (status: string) => {
  const code = parseInt(status);
  if (code >= 200 && code < 300) return "success";
  if (code >= 300 && code < 400) return "warning";
  if (code >= 400 && code < 500) return "danger";
  if (code >= 500) return "danger";
  return "info";
};

// 获取状态码说明
const getStatusDescription = (status: string) => {
  const statusMap: { [key: string]: string } = {
    "200": "请求成功",
    "301": "永久重定向",
    "302": "临时重定向",
    "400": "错误请求",
    "401": "未授权",
    "403": "禁止访问",
    "404": "未找到",
    "500": "服务器内部错误",
    "502": "网关错误",
    "503": "服务不可用",
    "504": "网关超时",
  };
  return statusMap[status] || "未知状态";
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

.url-input {
  flex: 1;
  min-width: 200px;
}

.check-btn {
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

  .header h1 {
    font-size: 20px;
  }

  .input-section {
    flex-direction: column;
  }
}
</style>
