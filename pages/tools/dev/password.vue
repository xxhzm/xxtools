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
        <h1>{{ title }}</h1>
        <p class="subtitle">{{ description }}</p>
      </div>
    </div>

    <div class="tool-content">
      <div class="input-section">
        <el-form :model="form" label-width="100px">
          <el-form-item label="密码">
            <div class="password-input">
              <el-input
                v-model="form.password"
                :type="showPassword ? 'text' : 'password'"
                placeholder="请输入密码"
                @input="checkPassword"
              >
                <template #suffix>
                  <el-icon
                    class="cursor-pointer"
                    @click="showPassword = !showPassword"
                  >
                    <View v-if="showPassword" />
                    <Hide v-else />
                  </el-icon>
                </template>
              </el-input>
              <el-button @click="generatePassword">
                <el-icon><RefreshRight /></el-icon>
                生成强密码
              </el-button>
            </div>
          </el-form-item>
        </el-form>
      </div>

      <div class="result-section" v-if="form.password">
        <div class="score-section">
          <div class="score-header">
            <h3>密码强度评分</h3>
            <div class="score-value">{{ passwordScore }} / 100</div>
          </div>
          <el-progress
            :percentage="passwordScore"
            :color="strengthColor"
            :status="strengthStatus"
          />
          <div class="strength-label">{{ strengthLabel }}</div>
        </div>

        <div class="analysis-section">
          <h3>复杂度分析</h3>
          <div class="check-list">
            <div
              v-for="(item, index) in checkResults"
              :key="index"
              class="check-item"
              :class="{ passed: item.passed }"
            >
              <el-icon>
                <CircleCheck v-if="item.passed" />
                <CircleClose v-else />
              </el-icon>
              <span>{{ item.label }}</span>
            </div>
          </div>
        </div>

        <div class="suggestions-section">
          <h3>安全建议</h3>
          <ul class="suggestions-list">
            <li v-for="(suggestion, index) in suggestions" :key="index">
              {{ suggestion }}
            </li>
          </ul>
        </div>
      </div>

      <div class="empty-state" v-else>
        <el-icon><Lock /></el-icon>
        <p>请输入密码进行强度检测</p>
      </div>
    </div>

    <div class="usage-guide">
      <h2>使用说明</h2>
      <ol>
        <li>
          输入需要检测的密码，或点击"生成强密码"按钮自动生成一个安全的密码
        </li>
        <li>
          系统将自动分析：
          <ul>
            <li>密码强度评分（0-100分）</li>
            <li>密码复杂度（长度、字符类型等）</li>
            <li>安全建议</li>
          </ul>
        </li>
        <li>根据分析结果改进密码强度</li>
      </ol>
      <div class="note">
        <el-icon><InfoFilled /></el-icon>
        <span>注意：密码不会被保存或传输，所有分析都在本地完成</span>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive, computed } from "vue";
import { useRouter } from "vue-router";
import {
  ArrowLeft,
  Lock,
  View,
  Hide,
  RefreshRight,
  CircleCheck,
  CircleClose,
  InfoFilled,
} from "@element-plus/icons-vue";
import toolsData from "~/data/data.json";

const router = useRouter();
const toolInfo = toolsData.tools.find((tool) => tool.id === 41);

// 更新页面标题和描述
const title = toolInfo?.title || "密码强度检测";
const description = toolInfo?.description || "密码强度检测工具";

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

// 表单数据
const form = reactive({
  password: "",
});

const showPassword = ref(false);

// 密码强度评分 (0-100)
const passwordScore = ref(0);

// 强度状态
const strengthStatus = computed(() => {
  if (passwordScore.value >= 80) return "success";
  if (passwordScore.value >= 60) return "warning";
  return "exception";
});

// 强度颜色
const strengthColor = computed(() => {
  if (passwordScore.value >= 80) return "#67c23a";
  if (passwordScore.value >= 60) return "#e6a23c";
  return "#f56c6c";
});

// 强度标签
const strengthLabel = computed(() => {
  if (passwordScore.value >= 80) return "非常安全";
  if (passwordScore.value >= 60) return "基本安全";
  if (passwordScore.value >= 40) return "较弱";
  return "不安全";
});

// 检查结果
const checkResults = reactive([
  { label: "长度至少8位", passed: false },
  { label: "包含大写字母", passed: false },
  { label: "包含小写字母", passed: false },
  { label: "包含数字", passed: false },
  { label: "包含特殊字符", passed: false },
]);

// 安全建议
const suggestions = ref<string[]>([]);

// 检查密码强度
const checkPassword = () => {
  const password = form.password;
  let score = 0;
  suggestions.value = [];

  // 重置检查结果
  checkResults[0].passed = password.length >= 8;
  checkResults[1].passed = /[A-Z]/.test(password);
  checkResults[2].passed = /[a-z]/.test(password);
  checkResults[3].passed = /[0-9]/.test(password);
  checkResults[4].passed = /[^A-Za-z0-9]/.test(password);

  // 计算基础分数
  if (password.length >= 8) score += 20;
  if (password.length >= 12) score += 10;
  if (password.length >= 16) score += 10; // 额外奖励长密码
  if (/[A-Z]/.test(password)) score += 15;
  if (/[a-z]/.test(password)) score += 15;
  if (/[0-9]/.test(password)) score += 15;
  if (/[^A-Za-z0-9]/.test(password)) score += 15;

  // 复杂度奖励
  const types = [/[A-Z]/, /[a-z]/, /[0-9]/, /[^A-Za-z0-9]/].filter((regex) =>
    regex.test(password)
  ).length;
  if (types >= 4) score += 10; // 包含所有字符类型的额外奖励

  // 分布均匀性检查
  const charTypes = {
    uppercase: (password.match(/[A-Z]/g) || []).length,
    lowercase: (password.match(/[a-z]/g) || []).length,
    numbers: (password.match(/[0-9]/g) || []).length,
    special: (password.match(/[^A-Za-z0-9]/g) || []).length,
  };
  const isBalanced = Object.values(charTypes).every((count) => count >= 2);
  if (isBalanced) score += 10; // 字符分布均匀的额外奖励

  // 额外检查
  if (password.length < 8) {
    suggestions.value.push("建议密码长度至少8位");
  }
  if (!/[A-Z]/.test(password)) {
    suggestions.value.push("建议包含大写字母");
  }
  if (!/[a-z]/.test(password)) {
    suggestions.value.push("建议包含小写字母");
  }
  if (!/[0-9]/.test(password)) {
    suggestions.value.push("建议包含数字");
  }
  if (!/[^A-Za-z0-9]/.test(password)) {
    suggestions.value.push("建议包含特殊字符");
  }

  // 检查常见弱密码模式
  if (/^[0-9]+$/.test(password)) {
    score -= 30;
    suggestions.value.push("纯数字密码容易被破解");
  }
  if (/^[a-zA-Z]+$/.test(password)) {
    score -= 30;
    suggestions.value.push("纯字母密码容易被破解");
  }
  if (/(.)\1{2,}/.test(password)) {
    score -= 20;
    suggestions.value.push("避免使用重复的字符");
  }
  if (/^(?:abc|123|password|admin|qwerty)/i.test(password)) {
    score -= 30;
    suggestions.value.push("避免使用常见的密码模式");
  }

  // 确保分数在0-100范围内
  passwordScore.value = Math.max(0, Math.min(100, score));

  // 如果没有具体建议，添加一般性建议
  if (suggestions.value.length === 0) {
    if (passwordScore.value < 60) {
      suggestions.value.push("建议增加密码的复杂度");
    } else if (passwordScore.value < 80) {
      suggestions.value.push("密码强度尚可，但仍有提升空间");
    } else if (passwordScore.value < 100) {
      suggestions.value.push(
        "密码强度很好，可以考虑增加长度或复杂度以达到最高安全级别"
      );
    } else {
      suggestions.value.push("密码强度达到最高级别，请牢记并定期更换");
    }
  }
};

// 生成强密码
const generatePassword = () => {
  const length = 20; // 增加默认长度到20位
  const charset = {
    uppercase: "ABCDEFGHIJKLMNOPQRSTUVWXYZ",
    lowercase: "abcdefghijklmnopqrstuvwxyz",
    numbers: "0123456789",
    symbols: "!@#$%^&*()_+-=[]{}|;:,.<>?",
  };

  let password = "";
  // 确保每种字符都至少出现两次
  for (const type of Object.values(charset)) {
    for (let i = 0; i < 2; i++) {
      password += type[Math.floor(Math.random() * type.length)];
    }
  }

  // 填充剩余长度
  const allChars = Object.values(charset).join("");
  for (let i = password.length; i < length; i++) {
    password += allChars[Math.floor(Math.random() * allChars.length)];
  }

  // 打乱密码字符顺序
  password = password
    .split("")
    .sort(() => Math.random() - 0.5)
    .join("");

  form.password = password;
  checkPassword();
};
</script>

<style scoped>
.container {
  max-width: 1200px;
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
  background: var(--el-bg-color-page);
  border-radius: 8px;
  padding: 20px;
  margin-bottom: 24px;
}

.password-input {
  display: flex;
  gap: 16px;
  align-items: center;
}

.password-input .el-input {
  flex: 1;
}

.result-section {
  background: var(--el-bg-color-page);
  border-radius: 8px;
  padding: 20px;
}

.score-section {
  background: var(--el-bg-color);
  border-radius: 8px;
  padding: 16px;
  margin-bottom: 24px;
}

.score-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 16px;
}

.score-header h3 {
  margin: 0;
  font-size: 16px;
  color: var(--el-text-color-primary);
}

.score-value {
  font-size: 24px;
  font-weight: bold;
  color: var(--el-text-color-primary);
}

.strength-label {
  margin-top: 8px;
  text-align: center;
  color: var(--el-text-color-regular);
}

.analysis-section {
  background: var(--el-bg-color);
  border-radius: 8px;
  padding: 16px;
  margin-bottom: 24px;
}

.analysis-section h3 {
  margin: 0 0 16px 0;
  font-size: 16px;
  color: var(--el-text-color-primary);
}

.check-list {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.check-item {
  display: flex;
  align-items: center;
  gap: 8px;
  color: var(--el-text-color-regular);
}

.check-item.passed {
  color: var(--el-color-success);
}

.check-item .el-icon {
  font-size: 18px;
}

.check-item.passed .el-icon {
  color: var(--el-color-success);
}

.suggestions-section {
  background: var(--el-bg-color);
  border-radius: 8px;
  padding: 16px;
}

.suggestions-section h3 {
  margin: 0 0 16px 0;
  font-size: 16px;
  color: var(--el-text-color-primary);
}

.suggestions-list {
  margin: 0;
  padding-left: 20px;
  color: var(--el-text-color-regular);
}

.suggestions-list li {
  margin-bottom: 8px;
}

.empty-state {
  height: 200px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  color: var(--el-text-color-secondary);
  gap: 12px;
}

.empty-state .el-icon {
  font-size: 48px;
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
  margin-top: 16px;
  display: flex;
  align-items: center;
  gap: 8px;
  color: var(--el-text-color-secondary);
  font-size: 14px;
}

.note .el-icon {
  color: var(--el-color-warning);
}

.cursor-pointer {
  cursor: pointer;
}
</style>
