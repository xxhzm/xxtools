<template>
  <div class="container">
    <div class="nav-header">
      <el-button link @click="router.push('/')" class="back-btn">
        <el-icon><ArrowLeft /></el-icon>
        返回首页
      </el-button>
    </div>

    <div class="tool-header">
      <el-icon class="header-icon"><DataLine /></el-icon>
      <div class="header-text">
        <h1>进制转换</h1>
        <p class="subtitle">
          在线进制转换工具，支持二进制、八进制、十进制、十六进制互转
        </p>
      </div>
    </div>

    <div class="tool-content">
      <div class="input-section">
        <el-input
          v-model="inputValue"
          placeholder="输入需要转换的数值"
          :prefix-icon="Edit"
          clearable
          @input="handleInput"
        >
          <template #append>
            <el-select v-model="inputBase" style="width: 120px">
              <el-option label="二进制" value="2" />
              <el-option label="八进制" value="8" />
              <el-option label="十进制" value="10" />
              <el-option label="十六进制" value="16" />
            </el-select>
          </template>
        </el-input>
      </div>

      <div class="result-section" v-if="inputValue">
        <div class="result-card">
          <div class="result-item">
            <div class="base-label">二进制 (Binary)</div>
            <div class="base-value">
              <span class="value">{{ binaryResult }}</span>
              <el-button link type="primary" @click="copyText(binaryResult)">
                复制
              </el-button>
            </div>
          </div>
          <div class="result-item">
            <div class="base-label">八进制 (Octal)</div>
            <div class="base-value">
              <span class="value">{{ octalResult }}</span>
              <el-button link type="primary" @click="copyText(octalResult)">
                复制
              </el-button>
            </div>
          </div>
          <div class="result-item">
            <div class="base-label">十进制 (Decimal)</div>
            <div class="base-value">
              <span class="value">{{ decimalResult }}</span>
              <el-button link type="primary" @click="copyText(decimalResult)">
                复制
              </el-button>
            </div>
          </div>
          <div class="result-item">
            <div class="base-label">十六进制 (Hexadecimal)</div>
            <div class="base-value">
              <span class="value">{{ hexResult }}</span>
              <el-button link type="primary" @click="copyText(hexResult)">
                复制
              </el-button>
            </div>
          </div>
        </div>
      </div>

      <div class="error-message" v-if="errorMessage">
        <el-alert :title="errorMessage" type="error" show-icon />
      </div>
    </div>

    <div class="usage-guide">
      <h2>使用说明</h2>
      <ol>
        <li>在输入框中输入需要转换的数值</li>
        <li>选择输入数值的进制（二进制、八进制、十进制、十六进制）</li>
        <li>自动显示其他进制的转换结果</li>
        <li>点击复制按钮可快速复制转换结果</li>
      </ol>
      <div class="note">
        <el-icon><InfoFilled /></el-icon>
        <span>提示：十六进制数值请使用0-9和a-f（不区分大小写）</span>
      </div>
    </div>

    <!-- 添加一个隐藏的input用于复制 -->
    <input ref="copyInput" type="text" class="copy-input" />
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from "vue";
import { ElMessage } from "element-plus";
import { useRouter } from "vue-router";
import { DataLine, Edit, InfoFilled, ArrowLeft } from "@element-plus/icons-vue";
import toolsData from "~/data/data.json";

const router = useRouter();
const copyInput = ref<HTMLInputElement | null>(null);

// 从data.json导入工具信息
const toolInfo = toolsData.tools.find((tool) => tool.id === 17);

// 更新页面标题和描述
const title = toolInfo?.title || "进制转换";
const description = toolInfo?.description || "在线进制转换工具";

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

const inputValue = ref("");
const inputBase = ref("10");
const errorMessage = ref("");

// 验证输入值是否合法
const validateInput = (value: string, base: number) => {
  if (!value) return true;
  const pattern = {
    2: /^[01]+$/,
    8: /^[0-7]+$/,
    10: /^\d+$/,
    16: /^[0-9a-f]+$/i,
  }[base];
  return pattern?.test(value);
};

// 处理输入变化
const handleInput = () => {
  errorMessage.value = "";
  if (!inputValue.value) return;

  const base = parseInt(inputBase.value);
  if (!validateInput(inputValue.value, base)) {
    errorMessage.value = `请输入有效的${
      { 2: "二进制", 8: "八进制", 10: "十进制", 16: "十六进制" }[base]
    }数值`;
    return;
  }
};

// 计算各进制结果
const decimalValue = computed(() => {
  if (!inputValue.value || errorMessage.value) return "";
  try {
    return parseInt(inputValue.value, parseInt(inputBase.value)).toString();
  } catch {
    return "";
  }
});

const binaryResult = computed(() => {
  if (!decimalValue.value) return "";
  try {
    return parseInt(decimalValue.value).toString(2);
  } catch {
    return "";
  }
});

const octalResult = computed(() => {
  if (!decimalValue.value) return "";
  try {
    return parseInt(decimalValue.value).toString(8);
  } catch {
    return "";
  }
});

const decimalResult = computed(() => {
  return decimalValue.value;
});

const hexResult = computed(() => {
  if (!decimalValue.value) return "";
  try {
    return parseInt(decimalValue.value).toString(16).toLowerCase();
  } catch {
    return "";
  }
});

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

.result-card {
  background: var(--el-bg-color-page);
  border: 1px solid var(--el-border-color);
  border-radius: 8px;
  padding: 20px;
}

.result-item {
  display: flex;
  flex-direction: column;
  margin-bottom: 20px;
}

.result-item:last-child {
  margin-bottom: 0;
}

.base-label {
  color: var(--el-text-color-secondary);
  font-size: 14px;
  margin-bottom: 8px;
}

.base-value {
  display: flex;
  align-items: center;
  gap: 12px;
  background: var(--el-fill-color-light);
  padding: 12px;
  border-radius: 4px;
}

.value {
  font-family: monospace;
  color: var(--el-text-color-primary);
  font-size: 16px;
  flex: 1;
  word-break: break-all;
}

.error-message {
  margin-top: 16px;
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
}
</style>
