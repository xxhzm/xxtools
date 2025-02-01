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
        <h1>图片Base64</h1>
        <p class="subtitle">
          在线图片转Base64编码工具，支持本地图片转换，完全在浏览器中进行
        </p>
      </div>
    </div>

    <div class="tool-content">
      <div class="upload-section">
        <el-upload
          class="image-uploader"
          action="#"
          :auto-upload="false"
          :show-file-list="false"
          :on-change="handleImageChange"
          accept="image/*"
        >
          <div class="upload-area" :class="{ 'has-image': imageUrl }">
            <template v-if="imageUrl">
              <img :src="imageUrl" class="preview-image" />
              <div class="image-info">
                <div class="info-item">
                  <span class="label">文件名：</span>
                  <span class="value">{{ imageInfo.name }}</span>
                </div>
                <div class="info-item">
                  <span class="label">大小：</span>
                  <span class="value">{{
                    formatFileSize(imageInfo.size)
                  }}</span>
                </div>
                <div class="info-item">
                  <span class="label">类型：</span>
                  <span class="value">{{ imageInfo.type }}</span>
                </div>
              </div>
            </template>
            <template v-else>
              <el-icon class="upload-icon"><Plus /></el-icon>
              <span>点击或拖拽图片到此处</span>
            </template>
          </div>
        </el-upload>
      </div>

      <div v-if="base64Result" class="result-section">
        <div class="result-header">
          <span class="result-label">Base64编码结果：</span>
          <div class="result-actions">
            <el-button type="primary" link @click="copyResult">
              <el-icon><DocumentCopy /></el-icon>
              复制结果
            </el-button>
          </div>
        </div>

        <el-input
          v-model="base64Result"
          type="textarea"
          :rows="6"
          readonly
          class="base64-result"
        />

        <div class="usage-tips">
          <div class="tip-item">
            <span class="label">HTML使用：</span>
            <el-tag size="small" @click="copyHtmlUsage">
              {{ htmlUsage }}
            </el-tag>
          </div>
          <div class="tip-item">
            <span class="label">CSS使用：</span>
            <el-tag size="small" @click="copyCssUsage">
              {{ cssUsage }}
            </el-tag>
          </div>
        </div>
      </div>
    </div>

    <div class="usage-guide">
      <h2>使用说明</h2>
      <ol>
        <li>点击上传区域或将图片拖拽到上传区域</li>
        <li>图片会自动转换为Base64编码</li>
        <li>可以复制完整的Base64编码</li>
        <li>提供HTML和CSS中的使用示例</li>
      </ol>
      <div class="note">
        <el-icon><InfoFilled /></el-icon>
        <span>提示：图片转换完全在本地进行，不会上传到服务器</span>
      </div>
    </div>

    <!-- 添加一个隐藏的input用于复制 -->
    <input ref="copyInput" type="text" class="copy-input" />
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from "vue";
import { useRouter } from "vue-router";
import {
  ArrowLeft,
  Picture,
  Plus,
  DocumentCopy,
  InfoFilled,
} from "@element-plus/icons-vue";
import { ElMessage } from "element-plus";
import type { UploadFile } from "element-plus";
import toolsData from "~/data/data.json";

const router = useRouter();
const copyInput = ref<HTMLInputElement | null>(null);

// 从data.json导入工具信息
const toolInfo = toolsData.tools.find((tool) => tool.id === 21);

// 更新页面标题和描述
const title = toolInfo?.title || "图片Base64";
const description = toolInfo?.description || "在线图片转Base64编码工具";

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

const imageUrl = ref("");
const base64Result = ref("");
const imageInfo = ref<{ name: string; size: number; type: string }>({
  name: "",
  size: 0,
  type: "",
});

// 处理图片选择
const handleImageChange = (uploadFile: UploadFile) => {
  const file = uploadFile.raw;
  if (!file) return;

  // 更新图片信息
  imageInfo.value = {
    name: file.name,
    size: file.size,
    type: file.type,
  };

  // 创建本地预览URL
  imageUrl.value = URL.createObjectURL(file);

  // 转换为Base64
  const reader = new FileReader();
  reader.onload = (e) => {
    if (e.target?.result) {
      base64Result.value = e.target.result as string;
    }
  };
  reader.readAsDataURL(file);
};

// 复制文本
const copyText = (text: string) => {
  if (!copyInput.value) return;
  copyInput.value.value = text;
  copyInput.value.select();
  try {
    document.execCommand("copy");
    ElMessage.success("复制成功");
  } catch (err) {
    ElMessage.error("复制失败，请手动复制");
  }
  copyInput.value.value = "";
};

// 复制Base64结果
const copyResult = () => {
  if (base64Result.value) {
    copyText(base64Result.value);
  }
};

// 在script部分添加计算属性
const htmlUsage = computed(() => {
  return `<img src="[Base64编码]" />`;
});

const cssUsage = computed(() => {
  return `background-image: url('[Base64编码]');`;
});

// 修改复制函数
const copyHtmlUsage = () => {
  if (base64Result.value) {
    copyText(`<img src="${base64Result.value}" />`);
  }
};

const copyCssUsage = () => {
  if (base64Result.value) {
    copyText(`background-image: url('${base64Result.value}');`);
  }
};

// 格式化文件大小
const formatFileSize = (size: number) => {
  if (size < 1024) return size + " B";
  if (size < 1024 * 1024) return (size / 1024).toFixed(2) + " KB";
  if (size < 1024 * 1024 * 1024)
    return (size / (1024 * 1024)).toFixed(2) + " MB";
  return (size / (1024 * 1024 * 1024)).toFixed(2) + " GB";
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

.upload-area {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  border: 2px dashed var(--el-border-color);
  border-radius: 8px;
  padding: 40px;
  cursor: pointer;
  transition: all 0.3s;
}

.upload-area:hover {
  border-color: var(--el-color-primary);
}

.upload-area.has-image {
  padding: 20px;
}

.upload-icon {
  font-size: 48px;
  color: var(--el-text-color-secondary);
  margin-bottom: 16px;
}

.preview-image {
  max-width: 100%;
  max-height: 300px;
  object-fit: contain;
  margin-bottom: 16px;
}

.image-info {
  width: 100%;
}

.info-item {
  display: flex;
  margin-bottom: 8px;
}

.info-item:last-child {
  margin-bottom: 0;
}

.info-item .label {
  width: 80px;
  color: var(--el-text-color-regular);
}

.info-item .value {
  color: var(--el-text-color-primary);
}

.result-section {
  margin-top: 24px;
}

.result-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 12px;
}

.result-label {
  font-size: 14px;
  color: var(--el-text-color-regular);
}

.base64-result {
  font-family: monospace;
  margin-bottom: 16px;
}

.usage-tips {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.tip-item {
  display: flex;
  align-items: center;
  gap: 8px;
}

.tip-item .el-tag {
  cursor: pointer;
  font-family: monospace;
  max-width: 100%;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  padding: 4px 8px;
}

.tip-item .el-tag:hover {
  background-color: var(--el-color-primary-light-8);
}

.tip-item .label {
  color: var(--el-text-color-regular);
  min-width: 80px;
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

.copy-input {
  position: absolute;
  top: -9999px;
  left: -9999px;
  opacity: 0;
  pointer-events: none;
}

@media (max-width: 768px) {
  .container {
    padding: 15px;
  }

  .upload-area {
    padding: 20px;
  }

  .result-header {
    flex-direction: column;
    align-items: flex-start;
    gap: 12px;
  }
}
</style>
