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
        <h1>随机密码</h1>
        <p class="subtitle">生成安全的随机密码，支持自定义长度和字符类型</p>
      </div>
    </div>

    <div class="tool-content">
      <div class="options-section">
        <div class="length-option">
          <span class="option-label">密码长度：</span>
          <el-input-number
            v-model="passwordLength"
            :min="4"
            :max="32"
            size="default"
            @change="generatePassword"
          />
        </div>

        <div class="char-options">
          <el-checkbox v-model="options.uppercase" @change="generatePassword">
            大写字母 (A-Z)
          </el-checkbox>
          <el-checkbox v-model="options.lowercase" @change="generatePassword">
            小写字母 (a-z)
          </el-checkbox>
          <el-checkbox v-model="options.numbers" @change="generatePassword">
            数字 (0-9)
          </el-checkbox>
          <el-checkbox v-model="options.symbols" @change="generatePassword">
            特殊字符 (!@#$%^&*)
          </el-checkbox>
        </div>

        <el-button
          type="primary"
          class="generate-btn"
          @click="generatePassword"
        >
          生成新密码
        </el-button>
      </div>

      <div v-if="password" class="result-section">
        <div class="password-display">
          <el-input
            v-model="password"
            readonly
            :prefix-icon="Lock"
            class="password-input"
          >
            <template #append>
              <el-button @click="copyPassword">
                <el-icon><CopyDocument /></el-icon>
                复制
              </el-button>
            </template>
          </el-input>
        </div>
        <div class="password-strength" v-if="password">
          <div class="strength-label">密码强度：</div>
          <el-progress
            :percentage="strengthScore"
            :color="strengthColor"
            :format="strengthText"
            :stroke-width="10"
          />
        </div>
      </div>
    </div>

    <div class="usage-guide">
      <h2>使用说明</h2>
      <ol>
        <li>选择所需的密码长度（4-32位）</li>
        <li>勾选需要包含的字符类型</li>
        <li>点击"生成新密码"按钮</li>
        <li>点击"复制"按钮复制生成的密码</li>
      </ol>
      <div class="note">
        <el-icon><InfoFilled /></el-icon>
        <span>提示：建议同时使用多种字符类型，生成更安全的密码</span>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from "vue";
import { ElMessage } from "element-plus";
import { useRouter } from "vue-router";
import {
  Key,
  Lock,
  CopyDocument,
  InfoFilled,
  ArrowLeft,
} from "@element-plus/icons-vue";
import toolsData from "~/data/data.json";

const router = useRouter();
const password = ref("");
const passwordLength = ref(12);
const options = ref({
  uppercase: true,
  lowercase: true,
  numbers: true,
  symbols: false,
});

// 从data.json导入工具信息
const toolInfo = toolsData.tools.find((tool) => tool.id === 11);

// 更新页面标题和描述
const title = toolInfo?.title || "随机密码";
const description =
  toolInfo?.description || "生成安全的随机密码，支持自定义长度和字符类型";

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

const generatePassword = () => {
  const chars = {
    uppercase: "ABCDEFGHIJKLMNOPQRSTUVWXYZ",
    lowercase: "abcdefghijklmnopqrstuvwxyz",
    numbers: "0123456789",
    symbols: "!@#$%^&*",
  };

  let availableChars = "";
  let hasSelectedOption = false;

  // 检查是否至少选择了一个选项
  Object.entries(options.value).forEach(([key, value]) => {
    if (value) {
      availableChars += chars[key as keyof typeof chars];
      hasSelectedOption = true;
    }
  });

  if (!hasSelectedOption) {
    ElMessage.warning("请至少选择一种字符类型");
    return;
  }

  let result = "";
  const length = passwordLength.value;
  const charactersLength = availableChars.length;

  // 使用 crypto.getRandomValues 生成更安全的随机数
  const randomBuffer = new Uint32Array(length);
  crypto.getRandomValues(randomBuffer);

  for (let i = 0; i < length; i++) {
    result += availableChars.charAt(randomBuffer[i] % charactersLength);
  }

  password.value = result;
};

const copyPassword = async () => {
  if (!password.value) {
    ElMessage.warning("请先生成密码");
    return;
  }

  try {
    // 尝试使用现代 Clipboard API
    await navigator.clipboard.writeText(password.value);
    ElMessage.success("密码已复制到剪贴板");
  } catch (err) {
    // 如果 Clipboard API 失败，尝试使用传统方法
    try {
      const textarea = document.createElement("textarea");
      textarea.value = password.value;
      textarea.style.position = "fixed";
      textarea.style.opacity = "0";
      document.body.appendChild(textarea);
      textarea.select();

      const successful = document.execCommand("copy");
      document.body.removeChild(textarea);

      if (successful) {
        ElMessage.success("密码已复制到剪贴板");
      } else {
        throw new Error("复制失败");
      }
    } catch (fallbackErr) {
      ElMessage({
        message: "复制失败，请手动选择密码并使用 Ctrl+C（或 Command+C）复制",
        type: "error",
        duration: 5000,
      });
    }
  }
};

// 计算密码强度
const strengthScore = computed(() => {
  if (!password.value) return 0;

  let score = 0;
  const length = password.value.length;

  // 根据长度加分
  if (length >= 8) score += 20;
  if (length >= 12) score += 10;
  if (length >= 16) score += 10;

  // 根据字符类型加分
  if (/[A-Z]/.test(password.value)) score += 15;
  if (/[a-z]/.test(password.value)) score += 15;
  if (/[0-9]/.test(password.value)) score += 15;
  if (/[!@#$%^&*]/.test(password.value)) score += 15;

  return Math.min(score, 100);
});

// 密码强度颜色
const strengthColor = computed(() => {
  const score = strengthScore.value;
  if (score >= 80) return "#67C23A";
  if (score >= 60) return "#E6A23C";
  return "#F56C6C";
});

// 密码强度文本
const strengthText = computed(() => {
  const score = strengthScore.value;
  if (score >= 80) return () => "强";
  if (score >= 60) return () => "中";
  return () => "弱";
});

// 初始生成一个密码
generatePassword();
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

.options-section {
  margin-bottom: 24px;
}

.length-option {
  display: flex;
  align-items: center;
  gap: 12px;
  margin-bottom: 20px;
}

.option-label {
  color: var(--el-text-color-regular);
  min-width: 80px;
}

.char-options {
  display: flex;
  flex-wrap: wrap;
  gap: 16px;
  margin-bottom: 20px;
}

.generate-btn {
  width: 100%;
}

.result-section {
  padding-top: 24px;
  border-top: 1px solid var(--el-border-color-light);
}

.password-display {
  margin-bottom: 20px;
}

.password-input {
  font-family: monospace;
  font-size: 16px;
}

.password-strength {
  display: flex;
  align-items: center;
  gap: 12px;
}

.strength-label {
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

@media (max-width: 768px) {
  .container {
    padding: 15px;
  }

  .char-options {
    flex-direction: column;
    gap: 12px;
  }

  .password-strength {
    flex-direction: column;
    align-items: flex-start;
  }

  .strength-label {
    margin-bottom: 8px;
  }
}
</style>
