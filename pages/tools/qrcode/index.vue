<template>
  <div class="container">
    <div class="nav-header">
      <el-button link @click="router.push('/')" class="back-btn">
        <el-icon><ArrowLeft /></el-icon>
        返回首页
      </el-button>
    </div>

    <div class="tool-header">
      <el-icon class="header-icon"><QrCode /></el-icon>
      <div class="header-text">
        <h1>在线二维码生成</h1>
        <p class="subtitle">快速生成二维码，支持文本、链接等内容</p>
      </div>
    </div>

    <div class="tool-content">
      <div class="input-section">
        <el-input
          v-model="text"
          type="textarea"
          :rows="4"
          placeholder="请输入需要生成二维码的内容，如文本、网址等"
          @input="handleInput"
        />
        <el-button
          type="primary"
          class="generate-btn"
          @click="generateQRCode"
          :loading="loading"
        >
          生成二维码
        </el-button>
      </div>

      <div v-if="qrcodeUrl" class="result-section">
        <div class="qrcode-wrapper">
          <img
            :src="qrcodeUrl"
            alt="生成的二维码"
            class="qrcode-image"
            @load="handleImageLoad"
            crossorigin="anonymous"
          />
          <el-button
            type="primary"
            @click="downloadQRCode"
            class="download-btn"
            :disabled="!imageLoaded"
          >
            <el-icon><Download /></el-icon>
            下载二维码
          </el-button>
        </div>
      </div>
    </div>

    <div class="usage-guide">
      <h2>使用说明</h2>
      <ol>
        <li>在输入框中输入需要生成二维码的内容（文本、网址等）</li>
        <li>点击"生成二维码"按钮</li>
        <li>等待二维码生成完成</li>
        <li>点击"下载二维码"保存图片</li>
      </ol>
      <div class="note">
        <el-icon><InfoFilled /></el-icon>
        <span>注意：生成的二维码图片会在一段时间后自动清理，请及时保存</span>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from "vue";
import { ElMessage } from "element-plus";
import { useRouter } from "vue-router";
import {
  Picture as QrCode,
  Download,
  InfoFilled,
  ArrowLeft,
} from "@element-plus/icons-vue";
import toolsData from "~/data/tools.json";

const router = useRouter();
const text = ref("");
const qrcodeUrl = ref("");
const loading = ref(false);
const imageLoaded = ref(false);

// 从data.json导入工具信息
const toolInfo = toolsData.tools.find((tool) => tool.id === 2);

// 更新页面标题和描述
const title = toolInfo?.title || "二维码生成";
const description =
  toolInfo?.description || "快速生成二维码，支持文本、链接等内容";

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

const handleInput = () => {
  qrcodeUrl.value = "";
  imageLoaded.value = false;
};

const handleImageLoad = () => {
  imageLoaded.value = true;
};

const generateQRCode = async () => {
  if (!text.value.trim()) {
    ElMessage.warning("请输入需要生成二维码的内容");
    return;
  }

  loading.value = true;
  imageLoaded.value = false;
  try {
    const encodedText = encodeURIComponent(text.value);
    const response = await fetch(
      `https://v2.xxapi.cn/api/qrcode?text=${encodedText}&return=json`
    );
    const data = await response.json();

    if (data.code === 200) {
      qrcodeUrl.value = data.data;
    } else {
      ElMessage.error("二维码生成失败，请稍后重试");
    }
  } catch (error) {
    ElMessage.error("网络错误，请稍后重试");
  } finally {
    loading.value = false;
  }
};

const downloadQRCode = async () => {
  if (!imageLoaded.value) {
    ElMessage.warning("请等待图片加载完成");
    return;
  }

  try {
    const response = await fetch(qrcodeUrl.value);
    const blob = await response.blob();
    const url = window.URL.createObjectURL(blob);
    const a = document.createElement("a");
    a.href = url;
    a.download = "qrcode.png";
    document.body.appendChild(a);
    a.click();
    document.body.removeChild(a);
    window.URL.revokeObjectURL(url);
  } catch (error) {
    ElMessage.error("下载失败，请重试");
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
}

.generate-btn {
  margin-top: 16px;
  width: 100%;
}

.result-section {
  display: flex;
  justify-content: center;
  padding-top: 24px;
  border-top: 1px solid var(--el-border-color-light);
}

.qrcode-wrapper {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 16px;
}

.qrcode-image {
  max-width: 200px;
  border: 1px solid var(--el-border-color);
  border-radius: 8px;
  padding: 8px;
  background: white;
}

.download-btn {
  display: flex;
  align-items: center;
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
</style>
