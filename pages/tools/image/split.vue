<template>
  <div class="container">
    <div class="nav-header">
      <el-button link @click="router.push('/')" class="back-btn">
        <el-icon><ArrowLeft /></el-icon>
        返回首页
      </el-button>
    </div>

    <div class="tool-header">
      <el-icon class="header-icon"><Crop /></el-icon>
      <div class="header-text">
        <h1>{{ title }}</h1>
        <p class="subtitle">{{ description }}</p>
      </div>
    </div>

    <div class="tool-content">
      <div class="upload-area" v-if="!imageUrl">
        <el-upload
          class="image-upload"
          drag
          :auto-upload="false"
          accept="image/*"
          :show-file-list="false"
          @change="handleImageChange"
        >
          <el-icon class="upload-icon"><Upload /></el-icon>
          <div class="upload-text">拖拽图片到此处或点击上传</div>
        </el-upload>
      </div>

      <div v-else class="image-editor">
        <div class="preview-area">
          <img :src="imageUrl" ref="previewImage" @load="handleImageLoad" />
          <div class="grid-overlay" :style="gridStyle"></div>
        </div>

        <div class="settings-panel">
          <el-form label-position="top">
            <el-form-item label="行数">
              <el-input-number
                v-model="rows"
                :min="1"
                :max="10"
                @change="updateGrid"
                controls-position="right"
              />
            </el-form-item>
            <el-form-item label="列数">
              <el-input-number
                v-model="cols"
                :min="1"
                :max="10"
                @change="updateGrid"
                controls-position="right"
              />
            </el-form-item>
            <el-form-item>
              <el-button
                type="primary"
                @click="splitImage"
                :loading="processing"
                class="full-width"
              >
                开始切割
              </el-button>
              <el-button @click="resetImage" class="full-width"
                >重新上传</el-button
              >
            </el-form-item>
          </el-form>
        </div>
      </div>

      <div class="result-section" v-if="splitImages.length > 0">
        <div class="section-title">
          <h3>切割结果</h3>
          <el-button
            type="primary"
            @click="downloadAllImages"
            :loading="downloading"
          >
            <el-icon><Download /></el-icon>
            打包下载全部
          </el-button>
        </div>
        <div class="split-images">
          <div
            v-for="(img, index) in splitImages"
            :key="index"
            class="split-item"
          >
            <img :src="img" />
            <el-button size="small" @click="downloadImage(img, index)" text>
              <el-icon><Download /></el-icon>
              下载
            </el-button>
          </div>
        </div>
      </div>
    </div>

    <div class="usage-guide">
      <h2>使用说明</h2>
      <ol>
        <li>上传需要切割的图片</li>
        <li>设置需要切割的行数和列数（1-10）</li>
        <li>点击"开始切割"按钮进行切割</li>
        <li>下载切割后的图片（支持单张下载或打包下载）</li>
      </ol>
      <div class="note">
        <el-icon><InfoFilled /></el-icon>
        <span>注意：建议上传清晰的图片以获得更好的切割效果</span>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from "vue";
import { useRouter } from "vue-router";
import {
  ArrowLeft,
  Upload,
  Download,
  InfoFilled,
  Crop,
} from "@element-plus/icons-vue";
import { ElMessage } from "element-plus";
import JSZip from "jszip";
import toolsData from "~/data/data.json";

const router = useRouter();
const toolInfo = toolsData.tools.find((tool) => tool.id === 27);

// 更新页面标题和描述
const title = toolInfo?.title || "图片切割";
const description = toolInfo?.description || "在线图片切割工具";

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
const previewImage = ref<HTMLImageElement | null>(null);
const rows = ref(3);
const cols = ref(3);
const processing = ref(false);
const downloading = ref(false);
const splitImages = ref<string[]>([]);

// 计算网格样式
const gridStyle = computed(() => {
  const rowPercent = 100 / rows.value;
  const colPercent = 100 / cols.value;
  return {
    backgroundImage: `linear-gradient(0deg, #000 1px, transparent 1px),
                     linear-gradient(90deg, #000 1px, transparent 1px)`,
    backgroundSize: `${colPercent}% ${rowPercent}%`,
    opacity: 0.2,
  };
});

// 处理图片上传
const handleImageChange = (file: any) => {
  if (!file.raw) return;
  const isImage = file.raw.type.startsWith("image/");
  if (!isImage) {
    ElMessage.error("请上传图片文件");
    return;
  }

  const reader = new FileReader();
  reader.onload = (e) => {
    imageUrl.value = e.target?.result as string;
  };
  reader.readAsDataURL(file.raw);
};

// 处理图片加载
const handleImageLoad = () => {
  updateGrid();
};

// 更新网格
const updateGrid = () => {
  // 网格更新逻辑
};

// 重置图片
const resetImage = () => {
  imageUrl.value = "";
  splitImages.value = [];
};

// 切割图片
const splitImage = async () => {
  if (!previewImage.value) return;

  processing.value = true;
  splitImages.value = [];

  try {
    const img = previewImage.value;
    const canvas = document.createElement("canvas");
    const ctx = canvas.getContext("2d");
    if (!ctx) throw new Error("无法创建canvas上下文");

    const width = img.naturalWidth;
    const height = img.naturalHeight;
    const cellWidth = width / cols.value;
    const cellHeight = height / rows.value;

    for (let row = 0; row < rows.value; row++) {
      for (let col = 0; col < cols.value; col++) {
        canvas.width = cellWidth;
        canvas.height = cellHeight;

        ctx.drawImage(
          img,
          col * cellWidth,
          row * cellHeight,
          cellWidth,
          cellHeight,
          0,
          0,
          cellWidth,
          cellHeight
        );

        splitImages.value.push(canvas.toDataURL("image/png"));
      }
    }

    ElMessage.success("切割完成");
  } catch (error) {
    console.error(error);
    ElMessage.error("切割失败，请重试");
  } finally {
    processing.value = false;
  }
};

// 下载单张图片
const downloadImage = (dataUrl: string, index: number) => {
  const link = document.createElement("a");
  link.href = dataUrl;
  link.download = `split_${index + 1}.png`;
  link.click();
};

// 打包下载所有图片
const downloadAllImages = async () => {
  if (splitImages.value.length === 0) return;

  downloading.value = true;
  try {
    const zip = new JSZip();

    splitImages.value.forEach((dataUrl, index) => {
      const data = dataUrl.split(",")[1];
      zip.file(`split_${index + 1}.png`, data, { base64: true });
    });

    const content = await zip.generateAsync({ type: "blob" });
    const link = document.createElement("a");
    link.href = URL.createObjectURL(content);
    link.download = "split_images.zip";
    link.click();

    ElMessage.success("打包下载完成");
  } catch (error) {
    console.error(error);
    ElMessage.error("打包下载失败，请重试");
  } finally {
    downloading.value = false;
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

.image-editor {
  display: flex;
  gap: 20px;
  margin-bottom: 24px;
}

.preview-area {
  flex: 1;
  position: relative;
  min-height: 300px;
  display: flex;
  align-items: center;
  justify-content: center;
  border: 1px solid var(--el-border-color);
  border-radius: 8px;
  overflow: hidden;
  background: var(--el-bg-color-page);
}

.preview-area img {
  max-width: 100%;
  max-height: 500px;
  object-fit: contain;
}

.grid-overlay {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  pointer-events: none;
}

.settings-panel {
  width: 280px;
  padding: 20px;
  background: var(--el-fill-color-light);
  border-radius: 8px;
}

.full-width {
  width: 100%;
  margin-bottom: 10px;
}

.result-section {
  padding-top: 24px;
  border-top: 1px solid var(--el-border-color-light);
}

.section-title {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 20px;
}

.section-title h3 {
  font-size: 16px;
  color: var(--el-text-color-primary);
  margin: 0;
}

.split-images {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(180px, 1fr));
  gap: 20px;
  margin: 20px 0;
}

.split-item {
  text-align: center;
  background: var(--el-bg-color-page);
  border: 1px solid var(--el-border-color);
  border-radius: 8px;
  padding: 12px;
}

.split-item img {
  width: 100%;
  height: 180px;
  object-fit: contain;
  margin-bottom: 10px;
  border-radius: 4px;
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
  margin-bottom: 8px;
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

.upload-icon {
  font-size: 48px;
  color: var(--el-text-color-secondary);
  margin-bottom: 10px;
}

.upload-text {
  color: var(--el-text-color-secondary);
}

@media (max-width: 768px) {
  .container {
    padding: 15px;
  }

  .image-editor {
    flex-direction: column;
  }

  .settings-panel {
    width: 100%;
  }
}
</style>
