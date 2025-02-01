<template>
  <div class="container">
    <div class="nav-header">
      <el-button link @click="router.push('/')" class="back-btn">
        <el-icon><ArrowLeft /></el-icon>
        返回首页
      </el-button>
    </div>

    <div class="tool-header">
      <el-icon class="header-icon"><Picture /></el-icon>
      <div class="header-text">
        <h1>网站图标</h1>
        <p class="subtitle">获取任意网站的favicon图标</p>
      </div>
    </div>

    <div class="tool-content">
      <div class="input-section">
        <el-input
          v-model="url"
          placeholder="请输入网址，如: https://xxapi.cn"
          :prefix-icon="Link"
          clearable
          @keyup.enter="getIco"
        >
          <template #append>
            <el-button @click="getIco" :loading="loading"> 获取图标 </el-button>
          </template>
        </el-input>
      </div>

      <div v-if="icoUrl" class="result-section">
        <div class="ico-preview">
          <div class="preview-card">
            <img
              :src="icoUrl"
              alt="网站图标"
              class="ico-image"
              @error="handleImageError"
            />
            <div class="ico-url">
              <el-input v-model="icoUrl" readonly size="small">
                <template #append>
                  <el-button @click="copyUrl">
                    <el-icon><CopyDocument /></el-icon>
                    复制
                  </el-button>
                </template>
              </el-input>
            </div>
          </div>
        </div>
      </div>
    </div>

    <div class="usage-guide">
      <h2>使用说明</h2>
      <ol>
        <li>输入需要获取图标的网站地址</li>
        <li>点击"获取图标"按钮</li>
        <li>等待图标加载完成</li>
        <li>点击"复制"按钮复制图标地址</li>
      </ol>
      <div class="note">
        <el-icon><InfoFilled /></el-icon>
        <span>提示：部分网站可能未设置图标或图标无法访问</span>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from "vue";
import { ElMessage } from "element-plus";
import { useRouter } from "vue-router";
import {
  Picture,
  Link,
  CopyDocument,
  InfoFilled,
  ArrowLeft,
} from "@element-plus/icons-vue";
import toolsData from "~/data/data.json";

const router = useRouter();
const url = ref("");
const icoUrl = ref("");
const loading = ref(false);

// 从data.json导入工具信息
const toolInfo = toolsData.tools.find((tool) => tool.id === 12);

// 更新页面标题和描述
const title = toolInfo?.title || "网站图标";
const description = toolInfo?.description || "获取任意网站的favicon图标";

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

const getIco = async () => {
  if (!url.value.trim()) {
    ElMessage.warning("请输入网址");
    return;
  }

  loading.value = true;
  icoUrl.value = "";

  try {
    const response = await fetch(
      `https://v2.xxapi.cn/api/ico?url=${encodeURIComponent(url.value)}`
    );
    const data = await response.json();

    if (data.code === 200 && data.data) {
      icoUrl.value = data.data;
      ElMessage.success("获取成功");
    } else {
      ElMessage.error(data.msg || "获取失败，请检查网址是否正确");
    }
  } catch (error) {
    ElMessage.error("网络错误，请稍后重试");
  } finally {
    loading.value = false;
  }
};

const handleImageError = () => {
  ElMessage.error("图标加载失败，可能是图标不存在或无法访问");
};

const copyUrl = async () => {
  if (!icoUrl.value) {
    ElMessage.warning("请先获取图标");
    return;
  }

  try {
    await navigator.clipboard.writeText(icoUrl.value);
    ElMessage.success("图标地址已复制到剪贴板");
  } catch (err) {
    try {
      const textarea = document.createElement("textarea");
      textarea.value = icoUrl.value;
      textarea.style.position = "fixed";
      textarea.style.opacity = "0";
      document.body.appendChild(textarea);
      textarea.select();

      const successful = document.execCommand("copy");
      document.body.removeChild(textarea);

      if (successful) {
        ElMessage.success("图标地址已复制到剪贴板");
      } else {
        throw new Error("复制失败");
      }
    } catch (fallbackErr) {
      ElMessage({
        message: "复制失败，请手动选择地址并使用 Ctrl+C（或 Command+C）复制",
        type: "error",
        duration: 5000,
      });
    }
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

.result-section {
  padding-top: 24px;
  border-top: 1px solid var(--el-border-color-light);
}

.ico-preview {
  display: flex;
  justify-content: center;
}

.preview-card {
  background: var(--el-bg-color-page);
  border-radius: 8px;
  padding: 20px;
  border: 1px solid var(--el-border-color);
  width: 100%;
  max-width: 400px;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 20px;
}

.ico-image {
  width: 64px;
  height: 64px;
  object-fit: contain;
  border: 1px solid var(--el-border-color-light);
  border-radius: 4px;
  padding: 4px;
  background: white;
}

.ico-url {
  width: 100%;
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
