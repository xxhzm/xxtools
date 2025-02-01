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
        <h1>图片水印</h1>
        <p class="subtitle">
          在线图片添加水印工具，支持文字水印和图片水印，完全在本地处理
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
              <div class="canvas-container" ref="canvasContainer">
                <canvas ref="canvas" class="preview-canvas"></canvas>
              </div>
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
                  <span class="label">尺寸：</span>
                  <span class="value"
                    >{{ imageInfo.width }} x {{ imageInfo.height }}</span
                  >
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

      <div v-if="imageUrl" class="watermark-settings">
        <div class="settings-section">
          <h3>水印设置</h3>
          <el-form :model="watermarkSettings" label-width="80px">
            <el-form-item label="水印类型">
              <el-radio-group
                v-model="watermarkSettings.type"
                @change="updateWatermark"
              >
                <el-radio-button label="text">文字水印</el-radio-button>
                <el-radio-button label="image">图片水印</el-radio-button>
              </el-radio-group>
            </el-form-item>

            <template v-if="watermarkSettings.type === 'text'">
              <el-form-item label="水印文字">
                <el-input
                  v-model="watermarkSettings.text"
                  placeholder="请输入水印文字"
                  @input="updateWatermark"
                />
              </el-form-item>
              <el-form-item label="字体大小">
                <el-slider
                  v-model="watermarkSettings.fontSize"
                  :min="12"
                  :max="72"
                  @change="updateWatermark"
                />
              </el-form-item>
              <el-form-item label="字体颜色">
                <el-color-picker
                  v-model="watermarkSettings.color"
                  show-alpha
                  @change="updateWatermark"
                />
              </el-form-item>
            </template>

            <template v-else>
              <el-form-item label="水印图片">
                <el-upload
                  class="watermark-uploader"
                  action="#"
                  :auto-upload="false"
                  :show-file-list="false"
                  :on-change="handleWatermarkImageChange"
                  accept="image/*"
                >
                  <img
                    v-if="watermarkImageUrl"
                    :src="watermarkImageUrl"
                    class="watermark-image"
                  />
                  <el-button v-else type="primary">选择水印图片</el-button>
                </el-upload>
              </el-form-item>
              <el-form-item label="透明度">
                <el-slider
                  v-model="watermarkSettings.opacity"
                  :min="0"
                  :max="100"
                  @change="updateWatermark"
                />
              </el-form-item>
            </template>

            <el-form-item label="位置">
              <el-select
                v-model="watermarkSettings.position"
                @change="updateWatermark"
              >
                <el-option label="左上角" value="top-left" />
                <el-option label="右上角" value="top-right" />
                <el-option label="左下角" value="bottom-left" />
                <el-option label="右下角" value="bottom-right" />
                <el-option label="居中" value="center" />
              </el-select>
            </el-form-item>

            <el-form-item label="边距">
              <el-slider
                v-model="watermarkSettings.margin"
                :min="0"
                :max="100"
                @change="updateWatermark"
              />
            </el-form-item>
          </el-form>
        </div>

        <div class="action-buttons">
          <el-button type="primary" @click="downloadImage">
            <el-icon><Download /></el-icon>
            下载图片
          </el-button>
          <el-button @click="resetImage">
            <el-icon><RefreshLeft /></el-icon>
            重置图片
          </el-button>
        </div>
      </div>
    </div>

    <div class="usage-guide">
      <h2>使用说明</h2>
      <ol>
        <li>点击上传区域或将图片拖拽到上传区域</li>
        <li>选择水印类型（文字或图片）</li>
        <li>设置水印的样式和位置</li>
        <li>点击下载按钮保存处理后的图片</li>
      </ol>
      <div class="note">
        <el-icon><InfoFilled /></el-icon>
        <span>提示：图片处理完全在本地进行，不会上传到服务器</span>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive, onMounted } from "vue";
import { useRouter } from "vue-router";
import {
  ArrowLeft,
  Picture,
  Plus,
  Download,
  RefreshLeft,
  InfoFilled,
} from "@element-plus/icons-vue";
import { ElMessage } from "element-plus";
import type { UploadFile } from "element-plus";
import toolsData from "~/data/data.json";

const router = useRouter();

// 从data.json导入工具信息
const toolInfo = toolsData.tools.find((tool) => tool.id === 22);

// 更新页面标题和描述
const title = toolInfo?.title || "图片水印";
const description = toolInfo?.description || "在线图片添加水印工具";

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

const canvas = ref<HTMLCanvasElement | null>(null);
const canvasContainer = ref<HTMLDivElement | null>(null);
const imageUrl = ref("");
const watermarkImageUrl = ref("");
const originalImage = ref<HTMLImageElement | null>(null);
const watermarkImage = ref<HTMLImageElement | null>(null);
const imageInfo = ref<{
  name: string;
  size: number;
  width: number;
  height: number;
}>({
  name: "",
  size: 0,
  width: 0,
  height: 0,
});

const watermarkSettings = reactive({
  type: "text",
  text: "水印文字",
  fontSize: 24,
  color: "rgba(255, 255, 255, 0.5)",
  position: "bottom-right",
  margin: 20,
  opacity: 50,
});

// 处理图片选择
const handleImageChange = (uploadFile: UploadFile) => {
  const file = uploadFile.raw;
  if (!file) return;

  const img = new Image();
  img.onload = () => {
    // 更新图片信息
    imageInfo.value = {
      name: file.name,
      size: file.size,
      width: img.width,
      height: img.height,
    };

    // 保存原始图片
    originalImage.value = img;

    // 初始化画布
    initCanvas();

    // 绘制水印
    updateWatermark();
  };

  img.src = URL.createObjectURL(file);
  imageUrl.value = img.src;
};

// 处理水印图片选择
const handleWatermarkImageChange = (uploadFile: UploadFile) => {
  const file = uploadFile.raw;
  if (!file) return;

  const img = new Image();
  img.onload = () => {
    watermarkImage.value = img;
    watermarkImageUrl.value = img.src;
    updateWatermark();
  };

  img.src = URL.createObjectURL(file);
};

// 初始化画布
const initCanvas = () => {
  if (!canvas.value || !originalImage.value) return;

  const ctx = canvas.value.getContext("2d");
  if (!ctx) return;

  // 设置画布尺寸
  canvas.value.width = originalImage.value.width;
  canvas.value.height = originalImage.value.height;

  // 绘制原始图片
  ctx.drawImage(originalImage.value, 0, 0);
};

// 更新水印
const updateWatermark = () => {
  if (!canvas.value || !originalImage.value) return;

  const ctx = canvas.value.getContext("2d");
  if (!ctx) return;

  // 重新绘制原始图片
  ctx.clearRect(0, 0, canvas.value.width, canvas.value.height);
  ctx.drawImage(originalImage.value, 0, 0);

  if (watermarkSettings.type === "text" && watermarkSettings.text) {
    // 绘制文字水印
    ctx.font = `${watermarkSettings.fontSize}px Arial`;
    ctx.fillStyle = watermarkSettings.color;
    ctx.textBaseline = "middle";

    const textWidth = ctx.measureText(watermarkSettings.text).width;
    const textHeight = watermarkSettings.fontSize;

    const position = calculatePosition(textWidth, textHeight);
    ctx.fillText(watermarkSettings.text, position.x, position.y);
  } else if (watermarkSettings.type === "image" && watermarkImage.value) {
    // 绘制图片水印
    ctx.globalAlpha = watermarkSettings.opacity / 100;
    const position = calculatePosition(
      watermarkImage.value.width,
      watermarkImage.value.height
    );
    ctx.drawImage(watermarkImage.value, position.x, position.y);
    ctx.globalAlpha = 1;
  }
};

// 计算水印位置
const calculatePosition = (width: number, height: number) => {
  if (!canvas.value) return { x: 0, y: 0 };

  const margin = watermarkSettings.margin;
  let x = 0;
  let y = 0;

  switch (watermarkSettings.position) {
    case "top-left":
      x = margin;
      y = margin + height / 2;
      break;
    case "top-right":
      x = canvas.value.width - width - margin;
      y = margin + height / 2;
      break;
    case "bottom-left":
      x = margin;
      y = canvas.value.height - height / 2 - margin;
      break;
    case "bottom-right":
      x = canvas.value.width - width - margin;
      y = canvas.value.height - height / 2 - margin;
      break;
    case "center":
      x = (canvas.value.width - width) / 2;
      y = canvas.value.height / 2;
      break;
  }

  return { x, y };
};

// 下载图片
const downloadImage = () => {
  if (!canvas.value) return;

  const link = document.createElement("a");
  link.download = `watermark_${imageInfo.value.name}`;
  link.href = canvas.value.toDataURL("image/png");
  link.click();
};

// 重置图片
const resetImage = () => {
  if (!canvas.value || !originalImage.value) return;

  const ctx = canvas.value.getContext("2d");
  if (!ctx) return;

  ctx.clearRect(0, 0, canvas.value.width, canvas.value.height);
  ctx.drawImage(originalImage.value, 0, 0);
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

.canvas-container {
  width: 100%;
  margin-bottom: 16px;
  display: flex;
  justify-content: center;
}

.preview-canvas {
  max-width: 100%;
  max-height: 400px;
  object-fit: contain;
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

.watermark-settings {
  margin-top: 24px;
  padding-top: 24px;
  border-top: 1px solid var(--el-border-color);
}

.settings-section {
  margin-bottom: 24px;
}

.settings-section h3 {
  font-size: 16px;
  margin-bottom: 16px;
  color: var(--el-text-color-primary);
}

.watermark-uploader {
  display: flex;
  justify-content: center;
}

.watermark-image {
  width: 100px;
  height: 100px;
  object-fit: contain;
  border: 1px solid var(--el-border-color);
  border-radius: 4px;
  cursor: pointer;
}

.action-buttons {
  display: flex;
  gap: 12px;
  justify-content: center;
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

  .upload-area {
    padding: 20px;
  }

  .action-buttons {
    flex-direction: column;
  }
}
</style>
