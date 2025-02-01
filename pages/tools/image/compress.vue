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
        <h1>{{ title }}</h1>
        <p class="subtitle">{{ description }}</p>
      </div>
    </div>

    <div class="tool-content">
      <div class="upload-section">
        <el-upload
          class="image-uploader"
          :show-file-list="false"
          :before-upload="beforeUpload"
          :on-change="handleChange"
          accept="image/*"
          drag
        >
          <div v-if="!imageUrl" class="upload-placeholder">
            <el-icon class="upload-icon"><Upload /></el-icon>
            <div class="upload-text">
              <span>拖拽图片到此处或点击上传</span>
              <p>支持jpg、png、webp等格式</p>
            </div>
          </div>
          <img v-else :src="imageUrl" class="preview-image" />
        </el-upload>
      </div>

      <div class="settings-section" v-if="imageUrl">
        <div class="settings-header">
          <h3>压缩设置</h3>
        </div>
        <el-form :model="settings" label-width="120px">
          <el-form-item label="压缩质量">
            <el-slider
              v-model="settings.quality"
              :min="10"
              :max="100"
              :step="1"
              :format-tooltip="(val) => `${val}%`"
              @change="handleCompress"
            />
          </el-form-item>
          <el-form-item label="最大宽度">
            <el-input-number
              v-model="settings.maxWidth"
              :min="100"
              :max="10000"
              :step="100"
              @change="handleCompress"
            />
          </el-form-item>
          <el-form-item label="输出格式">
            <el-select v-model="settings.format" @change="handleCompress">
              <el-option label="原格式" value="original" />
              <el-option label="JPEG" value="jpeg" />
              <el-option label="PNG" value="png" />
              <el-option label="WEBP" value="webp" />
            </el-select>
          </el-form-item>
        </el-form>
      </div>

      <div class="result-section" v-if="compressedImageUrl">
        <div class="result-header">
          <h3>压缩结果</h3>
        </div>
        <div class="result-content">
          <div class="image-comparison">
            <div class="original-image">
              <h4>原始图片</h4>
              <img :src="imageUrl" />
              <p>大小：{{ formatFileSize(originalSize) }}</p>
            </div>
            <div class="compressed-image">
              <h4>压缩后</h4>
              <img :src="compressedImageUrl" />
              <p>大小：{{ formatFileSize(compressedSize) }}</p>
            </div>
          </div>
          <div class="compression-info">
            <div class="info-item">
              <span class="label">压缩率：</span>
              <span class="value">{{ compressionRatio }}%</span>
            </div>
            <div class="info-item">
              <span class="label">节省空间：</span>
              <span class="value">{{ formatFileSize(savedSize) }}</span>
            </div>
          </div>
          <div class="download-section">
            <el-button type="primary" @click="downloadImage">
              <el-icon><Download /></el-icon>
              下载压缩图片
            </el-button>
          </div>
        </div>
      </div>
    </div>

    <div class="usage-guide">
      <h2>使用说明</h2>
      <ol>
        <li>将图片拖拽到上传区域或点击选择图片</li>
        <li>
          调整压缩设置：
          <ul>
            <li>压缩质量：影响图片清晰度和文件大小</li>
            <li>最大宽度：限制图片尺寸</li>
            <li>输出格式：选择压缩后的图片格式</li>
          </ul>
        </li>
        <li>预览压缩效果</li>
        <li>下载压缩后的图片</li>
      </ol>
      <div class="note">
        <el-icon><InfoFilled /></el-icon>
        <span>提示：建议根据实际需求调整压缩参数，平衡图片质量和文件大小</span>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive } from "vue";
import { useRouter } from "vue-router";
import {
  ArrowLeft,
  Picture,
  Upload,
  Download,
  InfoFilled,
} from "@element-plus/icons-vue";
import { ElMessage } from "element-plus";
import toolsData from "~/data/data.json";

const router = useRouter();
const toolInfo = toolsData.tools.find((tool) => tool.id === 34);

// 更新页面标题和描述
const title = toolInfo?.title || "图片压缩";
const description = toolInfo?.description || "在线图片压缩工具";

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

// 图片数据
const imageUrl = ref("");
const compressedImageUrl = ref("");
const originalSize = ref(0);
const compressedSize = ref(0);

// 压缩设置
const settings = reactive({
  quality: 80,
  maxWidth: 1920,
  format: "original",
});

// 计算压缩比例和节省空间
const compressionRatio = ref(0);
const savedSize = ref(0);

// 上传前检查
const beforeUpload = (file: File) => {
  // 定义允许的图片格式
  const allowedTypes = [
    "image/jpeg",
    "image/jpg",
    "image/png",
    "image/webp",
    "image/gif",
  ];
  const isAllowedType = allowedTypes.includes(file.type);
  const isLt10M = file.size / 1024 / 1024 < 10;

  if (!isAllowedType) {
    ElMessage.error("只能上传 JPG/JPEG/PNG/WEBP/GIF 格式的图片！");
    return false;
  }
  if (!isLt10M) {
    ElMessage.error("图片大小不能超过 10MB！");
    return false;
  }
  return true;
};

// 处理文件变化
const handleChange = (file: any) => {
  if (!file || !beforeUpload(file.raw)) return;

  const reader = new FileReader();
  reader.onload = (e) => {
    if (e.target?.result) {
      imageUrl.value = e.target.result as string;
      originalSize.value = file.raw.size;
      handleCompress();
    }
  };
  reader.readAsDataURL(file.raw);
};

// 压缩图片
const handleCompress = async () => {
  if (!imageUrl.value) return;

  const img = new Image();
  img.src = imageUrl.value;

  img.onload = () => {
    const canvas = document.createElement("canvas");
    let width = img.width;
    let height = img.height;

    // 调整尺寸
    if (width > settings.maxWidth) {
      const ratio = settings.maxWidth / width;
      width = settings.maxWidth;
      height = height * ratio;
    }

    canvas.width = width;
    canvas.height = height;

    const ctx = canvas.getContext("2d");
    if (!ctx) return;

    ctx.drawImage(img, 0, 0, width, height);

    // 获取输出格式
    const format =
      settings.format === "original"
        ? imageUrl.value.split(";")[0].split("/")[1]
        : settings.format;

    // 压缩并输出
    const quality = settings.quality / 100;
    const compressedDataUrl = canvas.toDataURL(`image/${format}`, quality);
    compressedImageUrl.value = compressedDataUrl;

    // 计算压缩后大小
    const base64Data = compressedDataUrl.split(",")[1];
    const compressedBytes = atob(base64Data).length;
    compressedSize.value = compressedBytes;

    // 计算压缩比例和节省空间
    // 统一使用 (原始大小 - 压缩后大小) / 原始大小 的公式
    // 当文件变大时，结果自然为负数，表示负的压缩率
    compressionRatio.value = Math.round(
      ((originalSize.value - compressedSize.value) / originalSize.value) * 100
    );
    savedSize.value = originalSize.value - compressedSize.value;
  };
};

// 格式化文件大小
const formatFileSize = (bytes: number) => {
  if (bytes === 0) return "0 B";
  const k = 1024;
  const sizes = ["B", "KB", "MB", "GB"];
  const i = Math.floor(Math.log(Math.abs(bytes)) / Math.log(k));
  const prefix = bytes < 0 ? "增加 " : "减少 ";
  return (
    prefix + (Math.abs(bytes) / Math.pow(k, i)).toFixed(2) + " " + sizes[i]
  );
};

// 下载压缩后的图片
const downloadImage = () => {
  if (!compressedImageUrl.value) return;

  const link = document.createElement("a");
  link.href = compressedImageUrl.value;
  link.download = `compressed_image.${
    settings.format === "original" ? "jpg" : settings.format
  }`;
  link.click();
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

.upload-section {
  margin-bottom: 24px;
}

.image-uploader {
  width: 100%;
}

.upload-placeholder {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 40px 0;
}

.upload-icon {
  font-size: 48px;
  color: var(--el-text-color-secondary);
  margin-bottom: 16px;
}

.upload-text {
  text-align: center;
}

.upload-text span {
  font-size: 16px;
  color: var(--el-text-color-primary);
}

.upload-text p {
  margin: 8px 0 0;
  font-size: 14px;
  color: var(--el-text-color-secondary);
}

.preview-image {
  max-width: 100%;
  max-height: 300px;
  object-fit: contain;
}

.settings-section {
  margin-top: 24px;
  padding-top: 24px;
  border-top: 1px solid var(--el-border-color-light);
}

.settings-header {
  margin-bottom: 24px;
}

.settings-header h3 {
  font-size: 18px;
  color: var(--el-text-color-primary);
  margin: 0;
}

.result-section {
  margin-top: 24px;
  padding-top: 24px;
  border-top: 1px solid var(--el-border-color-light);
}

.result-header {
  margin-bottom: 24px;
}

.result-header h3 {
  font-size: 18px;
  color: var(--el-text-color-primary);
  margin: 0;
}

.image-comparison {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 24px;
  margin-bottom: 24px;
}

.original-image,
.compressed-image {
  text-align: center;
}

.original-image h4,
.compressed-image h4 {
  font-size: 14px;
  color: var(--el-text-color-primary);
  margin: 0 0 12px;
}

.original-image img,
.compressed-image img {
  max-width: 100%;
  max-height: 200px;
  object-fit: contain;
  border-radius: 4px;
}

.original-image p,
.compressed-image p {
  margin: 8px 0 0;
  font-size: 14px;
  color: var(--el-text-color-secondary);
}

.compression-info {
  display: flex;
  justify-content: center;
  gap: 32px;
  margin-bottom: 24px;
}

.info-item {
  display: flex;
  align-items: center;
  gap: 8px;
}

.info-item .label {
  color: var(--el-text-color-secondary);
}

.info-item .value {
  color: var(--el-color-primary);
  font-weight: 500;
}

.download-section {
  text-align: center;
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
  line-height: 1.8;
}

.usage-guide li {
  margin-bottom: 12px;
}

.usage-guide ul {
  margin: 8px 0;
  padding-left: 20px;
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

  .image-comparison {
    grid-template-columns: 1fr;
  }

  .compression-info {
    flex-direction: column;
    align-items: center;
    gap: 16px;
  }
}
</style>
