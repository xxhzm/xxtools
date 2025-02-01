<template>
  <div class="container">
    <div class="nav-header">
      <el-button link @click="router.push('/')" class="back-btn">
        <el-icon><ArrowLeft /></el-icon>
        返回首页
      </el-button>
    </div>

    <div class="tool-header">
      <el-icon class="header-icon"><Edit /></el-icon>
      <div class="header-text">
        <h1>正则测试</h1>
        <p class="subtitle">
          在线正则表达式测试工具，支持实时匹配、替换和常用正则表达式示例
        </p>
      </div>
    </div>

    <div class="tool-content">
      <div class="regex-section">
        <div class="regex-input">
          <el-input
            v-model="pattern"
            placeholder="输入正则表达式，如: \d+"
            :prefix-icon="Edit"
            clearable
          >
            <template #append>
              <el-select
                v-model="flags"
                multiple
                collapse-tags
                placeholder="选择标志"
                style="width: 120px"
              >
                <el-option label="全局匹配 (g)" value="g" />
                <el-option label="忽略大小写 (i)" value="i" />
                <el-option label="多行匹配 (m)" value="m" />
              </el-select>
            </template>
          </el-input>
          <el-input
            v-model="replacement"
            placeholder="输入替换文本（可选）"
            :prefix-icon="Search"
            clearable
            class="mt-2"
          />
        </div>

        <div class="test-input mt-4">
          <el-input
            v-model="testText"
            type="textarea"
            :rows="6"
            placeholder="输入测试文本"
            resize="vertical"
          />
        </div>

        <div v-if="testText" class="result-section mt-4">
          <div class="result-header">
            <h3>匹配结果</h3>
            <div class="match-count" v-if="matches.length">
              找到 {{ matches.length }} 个匹配
            </div>
          </div>

          <div class="matches" v-if="matches.length">
            <div
              v-for="(match, index) in matches"
              :key="index"
              class="match-item"
            >
              <div class="match-content">
                <span class="match-index">#{{ index + 1 }}</span>
                <span class="match-text">{{ match[0] }}</span>
              </div>
              <div class="match-groups" v-if="match.length > 1">
                <div
                  v-for="(group, groupIndex) in match.slice(1)"
                  :key="groupIndex"
                  class="group-item"
                >
                  组 {{ groupIndex + 1 }}: {{ group }}
                </div>
              </div>
            </div>
          </div>

          <div v-if="replacement" class="replacement-result mt-4">
            <h3>替换结果</h3>
            <div class="replaced-text">{{ replacedText }}</div>
          </div>
        </div>
      </div>
    </div>

    <div class="examples-section mt-4">
      <h3>常用正则表达式示例</h3>
      <el-row :gutter="20">
        <el-col :span="12" v-for="example in examples" :key="example.name">
          <div class="example-card" @click="useExample(example)">
            <h4>{{ example.name }}</h4>
            <code>{{ example.pattern }}</code>
            <p>{{ example.description }}</p>
          </div>
        </el-col>
      </el-row>
    </div>

    <div class="usage-guide">
      <h2>使用说明</h2>
      <ol>
        <li>
          输入正则表达式，选择需要的标志（g-全局匹配，i-忽略大小写，m-多行匹配）
        </li>
        <li>如需替换功能，请输入替换文本</li>
        <li>在测试文本框中输入要匹配的文本</li>
        <li>查看匹配结果和替换结果</li>
        <li>可以点击示例快速使用常用正则表达式</li>
      </ol>
      <div class="note">
        <el-icon><InfoFilled /></el-icon>
        <span>提示：正则表达式不需要添加首尾的斜杠，工具会自动处理</span>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, watch } from "vue";
import { ElMessage } from "element-plus";
import { useRouter } from "vue-router";
import { Edit, Search, InfoFilled, ArrowLeft } from "@element-plus/icons-vue";
import toolsData from "~/data/data.json";

const router = useRouter();
const pattern = ref("");
const flags = ref<string[]>([]);
const testText = ref("");
const replacement = ref("");

// 从data.json导入工具信息
const toolInfo = toolsData.tools.find((tool) => tool.id === 15);

// 更新页面标题和描述
const title = toolInfo?.title || "正则测试";
const description = toolInfo?.description || "在线正则表达式测试工具";

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

// 常用正则表达式示例
const examples = [
  {
    name: "邮箱地址",
    pattern: "[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}",
    description: "匹配标准邮箱地址格式",
  },
  {
    name: "手机号码",
    pattern: "1[3-9]\\d{9}",
    description: "匹配中国大陆手机号码",
  },
  {
    name: "URL地址",
    pattern:
      "https?://[\\w-]+(\\.[\\w-]+)+([\\w.,@?^=%&:/~+#-]*[\\w@?^=%&/~+#-])?",
    description: "匹配标准URL地址",
  },
  {
    name: "身份证号",
    pattern:
      "[1-9]\\d{5}(19|20)\\d{2}(0[1-9]|1[0-2])(0[1-9]|[12]\\d|3[01])\\d{3}[0-9Xx]",
    description: "匹配18位身份证号码",
  },
];

// 计算匹配结果
const matches = computed(() => {
  if (!pattern.value || !testText.value) return [];
  try {
    const regex = new RegExp(pattern.value, flags.value.join(""));
    const matches = Array.from(testText.value.matchAll(regex));
    return matches;
  } catch (error) {
    return [];
  }
});

// 计算替换结果
const replacedText = computed(() => {
  if (!pattern.value || !testText.value || !replacement.value) return "";
  try {
    const regex = new RegExp(pattern.value, flags.value.join(""));
    return testText.value.replace(regex, replacement.value);
  } catch (error) {
    return "";
  }
});

// 使用示例
const useExample = (example: (typeof examples)[0]) => {
  pattern.value = example.pattern;
  if (!flags.value.includes("g")) {
    flags.value = ["g"];
  }
  ElMessage.success(`已应用"${example.name}"示例`);
};

// 监听正则表达式输入，捕获语法错误
watch(pattern, (newPattern) => {
  if (!newPattern) return;
  try {
    new RegExp(newPattern);
  } catch (error) {
    ElMessage.warning("正则表达式语法错误");
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

.mt-2 {
  margin-top: 8px;
}

.mt-4 {
  margin-top: 16px;
}

.result-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 16px;
}

.result-header h3 {
  margin: 0;
  font-size: 16px;
  color: var(--el-text-color-primary);
}

.match-count {
  color: var(--el-text-color-secondary);
  font-size: 14px;
}

.match-item {
  background: var(--el-bg-color-page);
  border: 1px solid var(--el-border-color);
  border-radius: 6px;
  padding: 12px;
  margin-bottom: 8px;
}

.match-content {
  display: flex;
  align-items: center;
  gap: 12px;
}

.match-index {
  color: var(--el-text-color-secondary);
  font-size: 14px;
}

.match-text {
  color: var(--el-color-primary);
  font-family: monospace;
  word-break: break-all;
}

.match-groups {
  margin-top: 8px;
  padding-top: 8px;
  border-top: 1px dashed var(--el-border-color);
}

.group-item {
  color: var(--el-text-color-regular);
  font-size: 14px;
  font-family: monospace;
  margin-bottom: 4px;
}

.replacement-result {
  border-top: 1px solid var(--el-border-color-light);
  padding-top: 16px;
}

.replacement-result h3 {
  margin: 0 0 12px;
  font-size: 16px;
  color: var(--el-text-color-primary);
}

.replaced-text {
  background: var(--el-bg-color-page);
  border: 1px solid var(--el-border-color);
  border-radius: 6px;
  padding: 12px;
  color: var(--el-text-color-regular);
  font-family: monospace;
  white-space: pre-wrap;
  word-break: break-all;
}

.examples-section {
  background: var(--el-bg-color);
  border-radius: 8px;
  padding: 24px;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.05);
  margin-bottom: 24px;
}

.examples-section h3 {
  margin: 0 0 16px;
  font-size: 18px;
  color: var(--el-text-color-primary);
}

.example-card {
  background: var(--el-bg-color-page);
  border: 1px solid var(--el-border-color);
  border-radius: 6px;
  padding: 16px;
  margin-bottom: 16px;
  cursor: pointer;
  transition: all 0.3s;
}

.example-card:hover {
  border-color: var(--el-color-primary);
  transform: translateY(-2px);
}

.example-card h4 {
  margin: 0 0 8px;
  color: var(--el-text-color-primary);
}

.example-card code {
  display: block;
  background: var(--el-fill-color-light);
  padding: 8px;
  border-radius: 4px;
  color: var(--el-text-color-regular);
  font-family: monospace;
  margin-bottom: 8px;
  word-break: break-all;
}

.example-card p {
  margin: 0;
  color: var(--el-text-color-secondary);
  font-size: 14px;
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

  .example-card {
    margin-bottom: 12px;
  }
}
</style>
