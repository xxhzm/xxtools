<template>
  <div class="container">
    <div class="nav-header">
      <el-button link @click="router.push('/')" class="back-btn">
        <el-icon><ArrowLeft /></el-icon>
        返回首页
      </el-button>
    </div>

    <div class="tool-header">
      <el-icon class="header-icon"><Brush /></el-icon>
      <div class="header-text">
        <h1>颜色转换</h1>
        <p class="subtitle">16进制颜色代码与RGB颜色值互相转换</p>
      </div>
    </div>

    <div class="tool-content">
      <div class="color-section">
        <div class="hex-input">
          <el-input
            v-model="hexColor"
            placeholder="输入16进制颜色值 (例如: #FF0000)"
            :prefix-icon="Edit"
            clearable
            @input="hexToRgb"
          >
            <template #prepend>#</template>
          </el-input>
        </div>

        <div class="rgb-input">
          <div class="rgb-inputs">
            <el-input-number
              v-model="rgbColor.r"
              :min="0"
              :max="255"
              placeholder="R"
              @change="rgbToHex"
            />
            <el-input-number
              v-model="rgbColor.g"
              :min="0"
              :max="255"
              placeholder="G"
              @change="rgbToHex"
            />
            <el-input-number
              v-model="rgbColor.b"
              :min="0"
              :max="255"
              placeholder="B"
              @change="rgbToHex"
            />
          </div>
        </div>

        <div v-if="isValidColor" class="color-preview">
          <div
            class="preview-box"
            :style="{ backgroundColor: previewColor }"
          ></div>
          <div class="color-values">
            <p>HEX: {{ displayHexColor }}</p>
            <p>RGB: {{ displayRgbColor }}</p>
          </div>
        </div>
      </div>
    </div>

    <div class="usage-guide">
      <h2>使用说明</h2>
      <ol>
        <li>
          输入16进制颜色代码（如：FF0000）或RGB颜色值（如：R:255, G:0, B:0）
        </li>
        <li>系统会自动转换并显示对应的颜色值</li>
        <li>可以查看颜色预览效果</li>
      </ol>
      <div class="note">
        <el-icon><InfoFilled /></el-icon>
        <span>注意：16进制颜色值以#开头，RGB颜色值范围为0-255</span>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from "vue";
import { ElMessage } from "element-plus";
import { useRouter } from "vue-router";
import { Edit, Brush, InfoFilled, ArrowLeft } from "@element-plus/icons-vue";
import toolsData from "~/data/data.json";

const router = useRouter();
const hexColor = ref("");
const rgbColor = ref({ r: 0, g: 0, b: 0 });
const isValidColor = ref(false);

// 从data.json导入工具信息
const toolInfo = toolsData.tools.find((tool) => tool.id === 10);

// 更新页面标题和描述
const title = toolInfo?.title || "颜色转换";
const description =
  toolInfo?.description || "16进制颜色代码与RGB颜色值互相转换";

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

const displayHexColor = computed(() => {
  return hexColor.value ? `#${hexColor.value}` : "";
});

const displayRgbColor = computed(() => {
  return `rgb(${rgbColor.value.r}, ${rgbColor.value.g}, ${rgbColor.value.b})`;
});

const previewColor = computed(() => {
  return displayHexColor.value || displayRgbColor.value;
});

const hexToRgb = () => {
  const hex = hexColor.value.replace("#", "");
  if (hex.length !== 6) {
    isValidColor.value = false;
    return;
  }

  const validHex = /^[0-9A-Fa-f]{6}$/;
  if (!validHex.test(hex)) {
    isValidColor.value = false;
    return;
  }

  const r = parseInt(hex.substring(0, 2), 16);
  const g = parseInt(hex.substring(2, 4), 16);
  const b = parseInt(hex.substring(4, 6), 16);

  rgbColor.value = { r, g, b };
  isValidColor.value = true;
};

const rgbToHex = () => {
  const { r, g, b } = rgbColor.value;
  const toHex = (n: number) => {
    const hex = n.toString(16);
    return hex.length === 1 ? "0" + hex : hex;
  };

  hexColor.value = `${toHex(r)}${toHex(g)}${toHex(b)}`.toUpperCase();
  isValidColor.value = true;
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

.color-section {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.hex-input {
  width: 100%;
}

.rgb-inputs {
  display: flex;
  gap: 12px;
}

.rgb-inputs :deep(.el-input-number) {
  width: 120px;
}

.color-preview {
  display: flex;
  align-items: center;
  gap: 20px;
  margin-top: 20px;
  padding: 20px;
  background: var(--el-bg-color-page);
  border-radius: 8px;
  border: 1px solid var(--el-border-color);
}

.preview-box {
  width: 100px;
  height: 100px;
  border-radius: 8px;
  border: 1px solid var(--el-border-color);
}

.color-values {
  flex: 1;
}

.color-values p {
  margin: 8px 0;
  color: var(--el-text-color-regular);
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

  .rgb-inputs {
    flex-direction: column;
  }

  .rgb-inputs :deep(.el-input-number) {
    width: 100%;
  }

  .color-preview {
    flex-direction: column;
    text-align: center;
  }
}
</style>
