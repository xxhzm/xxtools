<template>
  <div class="container">
    <div class="nav-header">
      <el-button link @click="router.push('/')" class="back-btn">
        <el-icon><ArrowLeft /></el-icon>
        返回首页
      </el-button>
    </div>

    <div class="tool-header">
      <el-icon class="header-icon"><Lock /></el-icon>
      <div class="header-text">
        <h1>隐私保护水印</h1>
        <p class="subtitle">
          为图片添加半透明文字水印，保护图片版权，防止未经授权传播
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
            <el-form-item label="水印内容">
              <el-input
                v-model="watermarkSettings.text"
                placeholder="请输入水印内容（如版权信息、识别码等）"
                @input="updateWatermark"
              />
            </el-form-item>

            <el-form-item label="透明度">
              <el-slider
                v-model="watermarkSettings.transparency"
                :min="10"
                :max="90"
                @change="updateWatermark"
                :marks="{ 10: '高透明', 50: '中等', 90: '低透明' }"
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

            <el-form-item label="旋转角度">
              <el-slider
                v-model="watermarkSettings.rotation"
                :min="-45"
                :max="45"
                @change="updateWatermark"
              />
            </el-form-item>

            <el-form-item label="水印密度">
              <el-slider
                v-model="watermarkSettings.density"
                :min="1"
                :max="10"
                @change="updateWatermark"
                :marks="{ 1: '密集', 5: '中等', 10: '稀疏' }"
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
        <li>输入需要添加的水印内容</li>
        <li>调整透明度、字体大小和颜色</li>
        <li>点击下载按钮保存添加水印的图片</li>
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
  Lock,
  Plus,
  Download,
  RefreshLeft,
  InfoFilled,
} from "@element-plus/icons-vue";
import { ElMessage } from "element-plus";
import type { UploadFile } from "element-plus";
import toolsData from "~/data/data.json";
import { useHead } from "unhead";

const router = useRouter();

// 从data.json导入工具信息
const toolInfo = ref<any>(null);
// 查找工具信息
toolInfo.value = toolsData.tools.find(
  (tool) => tool.id === "privacy-watermark"
);

// 设置页面标题和元信息
const title = toolInfo.value?.title || "隐私保护水印";
const description =
  toolInfo.value?.description || "在线图片添加隐私保护水印工具";

useHead({
  title: `${title} - ${toolsData.subtitle}`,
  meta: [
    {
      name: "description",
      content: description,
    },
    {
      name: "keywords",
      content: toolInfo.value?.keywords.join(",") || "",
    },
  ],
});

// 图片相关状态
const imageUrl = ref("");
const originalImageUrl = ref("");
const canvas = ref<HTMLCanvasElement | null>(null);
const canvasContainer = ref<HTMLElement | null>(null);
const image = ref<HTMLImageElement | null>(null);
const imageInfo = reactive({
  name: "",
  size: 0,
  width: 0,
  height: 0,
});

// 水印设置
const watermarkSettings = reactive({
  text: "仅供XX办理使用",
  transparency: 20,
  fontSize: 12,
  color: "#000000",
  rotation: -45, // 默认-45度旋转
  density: 7, // 默认密度为7
});

// 处理图片上传
const handleImageChange = (file: UploadFile) => {
  if (!file) return;

  imageInfo.name = file.name;
  imageInfo.size = file.size || 0;

  const reader = new FileReader();
  reader.onload = (e) => {
    if (e.target?.result) {
      originalImageUrl.value = e.target.result as string;
      imageUrl.value = e.target.result as string;

      const img = new Image();
      img.onload = () => {
        imageInfo.width = img.width;
        imageInfo.height = img.height;
        image.value = img;
        initCanvas();
      };
      img.src = e.target.result as string;
    }
  };
  reader.readAsDataURL(file.raw!);
};

// 初始化画布
const initCanvas = () => {
  if (!canvas.value || !image.value || !canvasContainer.value) return;

  const ctx = canvas.value.getContext("2d");
  if (!ctx) return;

  // 设置画布尺寸
  canvas.value.width = image.value.width;
  canvas.value.height = image.value.height;

  // 清除画布并绘制图像
  ctx.clearRect(0, 0, canvas.value.width, canvas.value.height);
  ctx.drawImage(image.value, 0, 0);

  // 应用初始水印
  updateWatermark();
};

// 更新水印
const updateWatermark = () => {
  if (!canvas.value || !image.value) return;

  const ctx = canvas.value.getContext("2d");
  if (!ctx) return;

  // 重绘原始图像
  ctx.clearRect(0, 0, canvas.value.width, canvas.value.height);
  ctx.drawImage(image.value, 0, 0);

  // 如果水印文字为空，则不添加水印
  if (!watermarkSettings.text) return;

  applyWatermark(ctx);
};

// 应用半透明水印
const applyWatermark = (ctx: CanvasRenderingContext2D) => {
  const text = watermarkSettings.text;
  const canvasWidth = canvas.value!.width;
  const canvasHeight = canvas.value!.height;

  // 设置透明度
  const alpha = watermarkSettings.transparency / 100;

  // 保存当前上下文状态
  ctx.save();

  // 设置半透明
  ctx.globalAlpha = alpha;

  // 设置字体和颜色
  ctx.font = `${watermarkSettings.fontSize}px Arial`;
  ctx.fillStyle = watermarkSettings.color;

  // 计算文本宽度
  const textWidth = ctx.measureText(text).width;

  // 在计算旋转角度时需要转换为弧度
  const rotationRad = (watermarkSettings.rotation * Math.PI) / 180;

  // 计算水印间距，密度值直接影响间距
  // 密度值越大，间距越大（更稀疏）
  const spacingX =
    ((canvasWidth / (12 - watermarkSettings.density)) *
      watermarkSettings.density) /
    5;
  const spacingY =
    ((canvasHeight / (12 - watermarkSettings.density)) *
      watermarkSettings.density) /
    5;

  // 设置背景绘制模式
  ctx.save();

  // 创建一个较大的偏移，使旋转后的水印能够覆盖整个画布
  for (let y = -canvasHeight; y < canvasHeight * 2; y += spacingY) {
    for (let x = -canvasWidth; x < canvasWidth * 2; x += spacingX) {
      ctx.save();

      // 平移到水印位置
      ctx.translate(x, y);

      // 旋转文本
      ctx.rotate(rotationRad);

      // 绘制文本
      ctx.fillText(text, 0, 0);

      ctx.restore();
    }
  }

  ctx.restore();

  // 恢复上下文状态
  ctx.restore();
};

// 下载处理后的图片
const downloadImage = () => {
  if (!canvas.value) return;

  const link = document.createElement("a");
  link.download = `watermark_${imageInfo.name}`;
  link.href = canvas.value.toDataURL("image/png");
  link.click();

  ElMessage.success("图片下载成功");
};

// 重置图片
const resetImage = () => {
  if (!image.value || !canvas.value) return;

  imageUrl.value = originalImageUrl.value;

  const ctx = canvas.value.getContext("2d");
  if (ctx) {
    ctx.clearRect(0, 0, canvas.value.width, canvas.value.height);
    ctx.drawImage(image.value, 0, 0);
  }

  // 重置水印设置
  watermarkSettings.text = "仅供XX办理使用";
  watermarkSettings.transparency = 20;
  watermarkSettings.fontSize = 12;
  watermarkSettings.color = "#000000";
  watermarkSettings.rotation = -45; // 默认-45度
  watermarkSettings.density = 7; // 恢复默认密度

  // 应用默认水印
  updateWatermark();
};

// 格式化文件大小
const formatFileSize = (size: number): string => {
  if (size < 1024) {
    return size + " B";
  } else if (size < 1024 * 1024) {
    return (size / 1024).toFixed(2) + " KB";
  } else {
    return (size / (1024 * 1024)).toFixed(2) + " MB";
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
