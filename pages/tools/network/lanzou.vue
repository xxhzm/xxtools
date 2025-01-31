<template>
  <div class="container">
    <div class="nav-header">
      <el-button link @click="router.push('/')" class="back-btn">
        <el-icon><ArrowLeft /></el-icon>
        返回首页
      </el-button>
    </div>

    <div class="tool-header">
      <el-icon class="header-icon"><Link /></el-icon>
      <div class="header-text">
        <h1>蓝奏云解析</h1>
        <p class="subtitle">解析蓝奏云分享链接，获取直接下载地址</p>
      </div>
    </div>

    <div class="tool-content">
      <div class="input-section">
        <el-input
          v-model="url"
          placeholder="请输入蓝奏云分享链接，如: https://xxx.lanzouj.com/xxx"
          :prefix-icon="Link"
          clearable
          class="url-input"
        />
        <el-input
          v-model="password"
          placeholder="访问密码（如果有）"
          :prefix-icon="Lock"
          clearable
          class="password-input"
        />
        <el-button
          type="primary"
          class="parse-btn"
          @click="startParse"
          :loading="loading"
        >
          开始解析
        </el-button>
      </div>

      <div v-if="result" class="result-section">
        <div class="result-card">
          <div class="result-item">
            <span class="label">下载地址：</span>
            <div class="value-wrapper">
              <el-input v-model="result" readonly class="result-input">
                <template #append>
                  <el-button @click="copyLink">
                    <el-icon><DocumentCopy /></el-icon>
                    复制
                  </el-button>
                </template>
              </el-input>
              <el-button type="primary" @click="openLink" class="open-btn">
                <el-icon><Download /></el-icon>
                立即下载
              </el-button>
            </div>
          </div>
        </div>
      </div>
    </div>

    <div class="usage-guide">
      <h2>使用说明</h2>
      <ol>
        <li>在输入框中粘贴蓝奏云分享链接</li>
        <li>如果有访问密码，请在密码框中输入</li>
        <li>点击"开始解析"按钮</li>
        <li>获取直接下载地址后，可以复制或直接下载</li>
      </ol>
      <div class="note">
        <el-icon><InfoFilled /></el-icon>
        <span>注意：解析结果仅供临时使用，链接可能会在一段时间后失效</span>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from "vue";
import { ElMessage } from "element-plus";
import { useRouter } from "vue-router";
import {
  Link,
  Lock,
  InfoFilled,
  ArrowLeft,
  DocumentCopy,
  Download,
} from "@element-plus/icons-vue";
import toolsData from "~/data/data.json";

const router = useRouter();
const url = ref("");
const password = ref("");
const result = ref("");
const loading = ref(false);

// 从data.json导入工具信息
const toolInfo = toolsData.tools.find((tool) => tool.id === 5);

// 更新页面标题和描述
const title = toolInfo?.title || "蓝奏云解析";
const description =
  toolInfo?.description || "解析蓝奏云分享链接，获取直接下载地址";

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

const startParse = async () => {
  if (!url.value.trim()) {
    ElMessage.warning("请输入需要解析的蓝奏云链接");
    return;
  }

  loading.value = true;
  try {
    const apiUrl = new URL("https://v2.xxapi.cn/api/lanzou");
    apiUrl.searchParams.append("url", url.value);
    if (password.value) {
      apiUrl.searchParams.append("password", password.value);
    }

    const response = await fetch(apiUrl.toString());
    const data = await response.json();

    if (data.code === 200 && data.data) {
      result.value = data.data;
      ElMessage.success("解析成功");
    } else {
      ElMessage.error(data.msg || "解析失败，请检查链接和密码是否正确");
    }
  } catch (error) {
    ElMessage.error("网络错误，请稍后重试");
  } finally {
    loading.value = false;
  }
};

const copyLink = () => {
  try {
    // 创建临时输入框
    const input = document.createElement("input");
    input.value = result.value;
    document.body.appendChild(input);
    // 选中文本
    input.select();
    input.setSelectionRange(0, 9999);
    // 执行复制
    document.execCommand("copy");
    // 移除临时输入框
    document.body.removeChild(input);
    ElMessage.success("下载链接已复制到剪贴板");
  } catch (err) {
    ElMessage.error("复制失败，请手动复制");
  }
};

const openLink = () => {
  window.open(result.value, "_blank");
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

.password-input {
  width: 200px;
}

.parse-btn {
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
  line-height: 32px;
}

.value-wrapper {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.result-input {
  width: 100%;
}

.open-btn {
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
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

  .password-input {
    width: 100%;
  }
}
</style>
