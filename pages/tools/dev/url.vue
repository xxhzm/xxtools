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
        <h1>{{ title }}</h1>
        <p class="subtitle">{{ description }}</p>
      </div>
    </div>

    <div class="tool-content">
      <div class="operation-type">
        <el-radio-group v-model="operationType" @change="handleOperation">
          <el-radio-button label="encode">URL编码</el-radio-button>
          <el-radio-button label="decode">URL解码</el-radio-button>
        </el-radio-group>
      </div>

      <div class="editor-section">
        <div class="editor-header">
          <h3>输入文本</h3>
          <div class="actions">
            <el-button size="small" @click="clearInput">
              <el-icon><Delete /></el-icon>
              清空
            </el-button>
            <el-button size="small" @click="loadExample">
              <el-icon><DocumentAdd /></el-icon>
              加载示例
            </el-button>
          </div>
        </div>
        <el-input
          v-model="inputText"
          type="textarea"
          :rows="8"
          :placeholder="inputPlaceholder"
          @input="handleOperation"
        />
      </div>

      <div class="options-section">
        <el-form :model="options" label-width="120px" size="small">
          <el-form-item label="编码方式">
            <el-radio-group v-model="options.mode" @change="handleOperation">
              <el-radio label="encodeURI">encodeURI</el-radio>
              <el-radio label="encodeURIComponent">encodeURIComponent</el-radio>
            </el-radio-group>
          </el-form-item>
        </el-form>
      </div>

      <div class="result-section">
        <div class="result-header">
          <h3>处理结果</h3>
          <div class="actions">
            <el-button size="small" type="primary" @click="copyResult">
              <el-icon><DocumentCopy /></el-icon>
              复制结果
            </el-button>
          </div>
        </div>
        <el-input
          v-model="outputText"
          type="textarea"
          :rows="8"
          readonly
          :placeholder="outputPlaceholder"
        />
      </div>
    </div>

    <div class="usage-guide">
      <h2>使用说明</h2>
      <ol>
        <li>选择需要进行的操作（编码/解码）</li>
        <li>
          选择编码方式：
          <ul>
            <li>
              <strong>encodeURI</strong>：用于编码完整的URL，不会编码
              <code>/ ? : @ & = + $ ,</code> 等字符
            </li>
            <li>
              <strong>encodeURIComponent</strong
              >：用于编码URL参数，会编码所有特殊字符
            </li>
          </ul>
        </li>
        <li>输入需要处理的文本</li>
        <li>复制处理结果使用</li>
      </ol>
      <div class="note">
        <el-icon><InfoFilled /></el-icon>
        <span
          >提示：建议对URL参数使用encodeURIComponent，对完整URL使用encodeURI</span
        >
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
  Link,
  Delete,
  DocumentAdd,
  DocumentCopy,
  InfoFilled,
} from "@element-plus/icons-vue";
import { ElMessage } from "element-plus";
import toolsData from "~/data/data.json";

const router = useRouter();
const copyInput = ref<HTMLInputElement | null>(null);
const toolInfo = toolsData.tools.find((tool) => tool.id === 28);

// 更新页面标题和描述
const title = toolInfo?.title || "URL编码";
const description = toolInfo?.description || "在线URL编码解码工具";

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

const operationType = ref<"encode" | "decode">("encode");
const inputText = ref("");
const outputText = ref("");
const options = ref({
  mode: "encodeURI",
});

// 计算属性：输入框提示文本
const inputPlaceholder = computed(() => {
  return operationType.value === "encode"
    ? "请输入需要编码的文本..."
    : "请输入需要解码的文本...";
});

// 计算属性：输出框提示文本
const outputPlaceholder = computed(() => {
  return operationType.value === "encode"
    ? "编码结果将显示在这里..."
    : "解码结果将显示在这里...";
});

// 处理编码/解码操作
const handleOperation = () => {
  if (!inputText.value) {
    outputText.value = "";
    return;
  }

  try {
    if (operationType.value === "encode") {
      outputText.value =
        options.value.mode === "encodeURI"
          ? encodeURI(inputText.value)
          : encodeURIComponent(inputText.value);
    } else {
      outputText.value = decodeURI(inputText.value);
    }
  } catch (error) {
    console.error(error);
    ElMessage.error("处理失败，请检查输入是否正确");
  }
};

// 清空输入
const clearInput = () => {
  inputText.value = "";
  outputText.value = "";
};

// 加载示例
const loadExample = () => {
  if (operationType.value === "encode") {
    inputText.value = "https://example.com/path?name=测试&type=示例";
  } else {
    inputText.value =
      "https%3A%2F%2Fexample.com%2Fpath%3Fname%3D%E6%B5%8B%E8%AF%95%26type%3D%E7%A4%BA%E4%BE%8B";
  }
  handleOperation();
};

// 复制结果
const copyResult = () => {
  if (!outputText.value) return;
  if (!copyInput.value) return;

  copyInput.value.value = outputText.value;
  copyInput.value.select();
  try {
    document.execCommand("copy");
    ElMessage.success("复制成功");
  } catch (err) {
    ElMessage.error("复制失败，请手动复制");
  }
  copyInput.value.value = "";
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

.operation-type {
  margin-bottom: 24px;
}

.editor-section,
.result-section {
  margin-bottom: 24px;
}

.editor-header,
.result-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 12px;
}

.editor-header h3,
.result-header h3 {
  font-size: 16px;
  color: var(--el-text-color-primary);
  margin: 0;
}

.actions {
  display: flex;
  gap: 8px;
}

.options-section {
  margin: 24px 0;
  padding: 16px;
  background: var(--el-fill-color-light);
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
  margin-bottom: 12px;
}

.usage-guide ul {
  margin: 8px 0;
  padding-left: 20px;
}

.usage-guide code {
  background: var(--el-fill-color-light);
  padding: 2px 6px;
  border-radius: 4px;
  font-family: monospace;
  color: var(--el-text-color-primary);
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

  .operation-type .el-radio-group {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    gap: 8px;
  }

  .editor-header,
  .result-header {
    flex-direction: column;
    align-items: flex-start;
    gap: 8px;
  }

  .actions {
    width: 100%;
    justify-content: flex-end;
  }
}
</style>
