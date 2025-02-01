<template>
  <div class="container">
    <div class="nav-header">
      <el-button link @click="router.push('/')" class="back-btn">
        <el-icon><ArrowLeft /></el-icon>
        返回首页
      </el-button>
    </div>

    <div class="tool-header">
      <el-icon class="header-icon"><Key /></el-icon>
      <div class="header-text">
        <h1>哈希计算</h1>
        <p class="subtitle">
          在线哈希计算工具，支持MD5、SHA1、SHA256、SHA512等多种算法
        </p>
      </div>
    </div>

    <div class="tool-content">
      <div class="input-section">
        <div class="algorithm-select">
          <el-radio-group v-model="algorithm">
            <el-radio-button label="md5">MD5</el-radio-button>
            <el-radio-button label="sha1">SHA1</el-radio-button>
            <el-radio-button label="sha256">SHA256</el-radio-button>
            <el-radio-button label="sha512">SHA512</el-radio-button>
          </el-radio-group>
        </div>

        <el-input
          v-model="inputText"
          type="textarea"
          :rows="6"
          placeholder="请输入要计算哈希值的文本"
          @input="calculateHash"
        />
      </div>

      <div class="result-section">
        <div class="result-header">
          <span class="result-label">计算结果：</span>
          <div class="result-actions">
            <el-switch
              v-model="uppercase"
              active-text="大写"
              inactive-text="小写"
              @change="calculateHash"
            />
            <el-button
              type="primary"
              link
              @click="copyResult"
              :disabled="!hashResult"
            >
              <el-icon><DocumentCopy /></el-icon>
              复制结果
            </el-button>
          </div>
        </div>

        <el-input
          v-model="hashResult"
          readonly
          :placeholder="resultPlaceholder"
          class="hash-result"
        >
          <template #append>
            <el-button @click="copyResult" :disabled="!hashResult">
              复制
            </el-button>
          </template>
        </el-input>
      </div>

      <div class="file-hash-section">
        <div class="section-title">
          <el-icon><Upload /></el-icon>
          <span>文件哈希计算</span>
        </div>
        <el-upload
          class="file-uploader"
          action="#"
          :auto-upload="false"
          :show-file-list="false"
          :on-change="handleFileChange"
        >
          <template #trigger>
            <el-button type="primary">
              <el-icon><FolderAdd /></el-icon>
              选择文件
            </el-button>
          </template>
          <template #tip>
            <div class="upload-tip">
              选择文件后自动计算哈希值，文件仅在本地处理，不会上传到服务器
            </div>
          </template>
        </el-upload>

        <div v-if="fileInfo" class="file-info">
          <div class="info-item">
            <span class="label">文件名：</span>
            <span class="value">{{ fileInfo.name }}</span>
          </div>
          <div class="info-item">
            <span class="label">大小：</span>
            <span class="value">{{ formatFileSize(fileInfo.size) }}</span>
          </div>
          <div class="info-item">
            <span class="label">哈希值：</span>
            <div class="hash-value">
              <el-input
                v-model="fileHashResult"
                readonly
                placeholder="计算中..."
              >
                <template #append>
                  <el-button @click="copyFileHash" :disabled="!fileHashResult">
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
        <li>选择要使用的哈希算法</li>
        <li>输入要计算哈希值的文本，或选择要计算的文件</li>
        <li>可以选择输出大写或小写结果</li>
        <li>点击复制按钮可以复制计算结果</li>
      </ol>
      <div class="note">
        <el-icon><InfoFilled /></el-icon>
        <span>提示：文件哈希计算完全在本地进行，不会上传文件</span>
      </div>
    </div>

    <!-- 添加一个隐藏的input用于复制 -->
    <input ref="copyInput" type="text" class="copy-input" />
  </div>
</template>

<script setup lang="ts">
import { ref, computed, watch } from "vue";
import { useRouter } from "vue-router";
import {
  ArrowLeft,
  Key,
  DocumentCopy,
  Upload,
  FolderAdd,
  InfoFilled,
} from "@element-plus/icons-vue";
import { ElMessage } from "element-plus";
import type { UploadFile, UploadFiles } from "element-plus";
import CryptoJS from "crypto-js";
import toolsData from "~/data/data.json";

const router = useRouter();
const copyInput = ref<HTMLInputElement | null>(null);

// 从data.json导入工具信息
const toolInfo = toolsData.tools.find((tool) => tool.id === 20);

// 更新页面标题和描述
const title = toolInfo?.title || "哈希计算";
const description = toolInfo?.description || "在线哈希计算工具";

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

const algorithm = ref("md5");
const inputText = ref("");
const hashResult = ref("");
const uppercase = ref(false);
const fileInfo = ref<File | null>(null);
const fileHashResult = ref("");

// 结果占位符
const resultPlaceholder = computed(() => {
  return inputText.value ? "计算中..." : "等待输入";
});

// 计算文本哈希
const calculateHash = () => {
  if (!inputText.value) {
    hashResult.value = "";
    return;
  }

  let result = "";
  switch (algorithm.value) {
    case "md5":
      result = CryptoJS.MD5(inputText.value).toString();
      break;
    case "sha1":
      result = CryptoJS.SHA1(inputText.value).toString();
      break;
    case "sha256":
      result = CryptoJS.SHA256(inputText.value).toString();
      break;
    case "sha512":
      result = CryptoJS.SHA512(inputText.value).toString();
      break;
  }

  hashResult.value = uppercase.value
    ? result.toUpperCase()
    : result.toLowerCase();
};

// 复制结果
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

const copyResult = () => {
  if (hashResult.value) {
    copyText(hashResult.value);
  }
};

const copyFileHash = () => {
  if (fileHashResult.value) {
    copyText(fileHashResult.value);
  }
};

// 处理文件选择
const handleFileChange = (uploadFile: UploadFile, uploadFiles: UploadFiles) => {
  const file = uploadFile.raw;
  if (!file) return;

  fileInfo.value = file;
  fileHashResult.value = "";

  const reader = new FileReader();
  reader.onload = (e) => {
    if (!e.target?.result) return;

    const wordArray = CryptoJS.lib.WordArray.create(
      e.target.result as ArrayBuffer
    );
    let result = "";

    switch (algorithm.value) {
      case "md5":
        result = CryptoJS.MD5(wordArray).toString();
        break;
      case "sha1":
        result = CryptoJS.SHA1(wordArray).toString();
        break;
      case "sha256":
        result = CryptoJS.SHA256(wordArray).toString();
        break;
      case "sha512":
        result = CryptoJS.SHA512(wordArray).toString();
        break;
    }

    fileHashResult.value = uppercase.value
      ? result.toUpperCase()
      : result.toLowerCase();
  };

  reader.readAsArrayBuffer(file);
};

// 格式化文件大小
const formatFileSize = (size: number) => {
  if (size < 1024) return size + " B";
  if (size < 1024 * 1024) return (size / 1024).toFixed(2) + " KB";
  if (size < 1024 * 1024 * 1024)
    return (size / (1024 * 1024)).toFixed(2) + " MB";
  return (size / (1024 * 1024 * 1024)).toFixed(2) + " GB";
};

// 监听算法变化
watch(algorithm, () => {
  calculateHash();
  if (fileInfo.value) {
    handleFileChange({ raw: fileInfo.value } as UploadFile, [] as UploadFiles);
  }
});
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

.algorithm-select {
  margin-bottom: 16px;
}

.result-section {
  margin-bottom: 24px;
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

.result-actions {
  display: flex;
  align-items: center;
  gap: 16px;
}

.hash-result {
  font-family: monospace;
}

.file-hash-section {
  border-top: 1px solid var(--el-border-color);
  padding-top: 24px;
}

.section-title {
  display: flex;
  align-items: center;
  gap: 8px;
  margin-bottom: 16px;
  color: var(--el-text-color-primary);
  font-size: 16px;
}

.file-uploader {
  margin-bottom: 16px;
}

.upload-tip {
  font-size: 12px;
  color: var(--el-text-color-secondary);
  margin-top: 8px;
}

.file-info {
  background: var(--el-fill-color-light);
  border-radius: 4px;
  padding: 16px;
}

.info-item {
  display: flex;
  margin-bottom: 12px;
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

.info-item .hash-value {
  flex: 1;
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

  .algorithm-select :deep(.el-radio-group) {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    gap: 8px;
  }

  .result-header {
    flex-direction: column;
    align-items: flex-start;
    gap: 12px;
  }

  .result-actions {
    width: 100%;
    justify-content: space-between;
  }
}
</style>
