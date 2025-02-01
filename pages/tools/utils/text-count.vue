<template>
  <div class="container">
    <div class="nav-header">
      <el-button link @click="router.push('/')" class="back-btn">
        <el-icon><ArrowLeft /></el-icon>
        返回首页
      </el-button>
    </div>

    <div class="tool-header">
      <el-icon class="header-icon"><Document /></el-icon>
      <div class="header-text">
        <h1>{{ title }}</h1>
        <p class="subtitle">{{ description }}</p>
      </div>
    </div>

    <div class="tool-content">
      <div class="editor-section">
        <div class="editor-container">
          <div class="editor-header">
            <h3>输入文本</h3>
            <div class="editor-actions">
              <el-button @click="clearInput" size="small">
                <el-icon><Delete /></el-icon>
                清空
              </el-button>
            </div>
          </div>
          <el-input
            v-model="textInput"
            type="textarea"
            :rows="12"
            placeholder="在此输入或粘贴要统计的文本..."
            @input="handleInput"
          />
        </div>

        <div class="stats-container">
          <div class="stats-header">
            <h3>统计结果</h3>
            <el-switch
              v-model="settings.excludeWhitespace"
              active-text="不计空格"
              @change="updateStats"
            />
          </div>

          <div class="stats-grid">
            <div class="stat-item">
              <div class="stat-label">总字符数</div>
              <div class="stat-value">{{ stats.totalChars }}</div>
            </div>
            <div class="stat-item">
              <div class="stat-label">中文字数</div>
              <div class="stat-value">{{ stats.chineseChars }}</div>
            </div>
            <div class="stat-item">
              <div class="stat-label">英文字母</div>
              <div class="stat-value">{{ stats.englishWords }}</div>
            </div>
            <div class="stat-item">
              <div class="stat-label">数字个数</div>
              <div class="stat-value">{{ stats.numbers }}</div>
            </div>
            <div class="stat-item">
              <div class="stat-label">标点符号</div>
              <div class="stat-value">{{ stats.punctuation }}</div>
            </div>
            <div class="stat-item">
              <div class="stat-label">空格数量</div>
              <div class="stat-value">{{ stats.spaces }}</div>
            </div>
            <div class="stat-item">
              <div class="stat-label">行数</div>
              <div class="stat-value">{{ stats.lines }}</div>
            </div>
            <div class="stat-item">
              <div class="stat-label">段落数</div>
              <div class="stat-value">{{ stats.paragraphs }}</div>
            </div>
          </div>

          <div class="detailed-stats">
            <el-collapse>
              <el-collapse-item title="详细信息" name="1">
                <div class="stat-details">
                  <div class="detail-item">
                    <span class="detail-label">去重字符数：</span>
                    <span class="detail-value">{{ stats.uniqueChars }}</span>
                  </div>
                  <div class="detail-item">
                    <span class="detail-label">平均词长：</span>
                    <span class="detail-value"
                      >{{ stats.avgWordLength.toFixed(1) }} 字符</span
                    >
                  </div>
                  <div class="detail-item">
                    <span class="detail-label">最长行字数：</span>
                    <span class="detail-value"
                      >{{ stats.maxLineLength }} 字符</span
                    >
                  </div>
                  <div class="detail-item">
                    <span class="detail-label">文本密度：</span>
                    <span class="detail-value"
                      >{{ stats.textDensity.toFixed(1) }}%</span
                    >
                  </div>
                </div>
              </el-collapse-item>
            </el-collapse>
          </div>
        </div>
      </div>
    </div>

    <div class="usage-guide">
      <h2>使用说明</h2>
      <ol>
        <li>在输入框中输入或粘贴需要统计的文本</li>
        <li>
          系统会自动统计并显示以下信息：
          <ul>
            <li>字符数：包括中文、英文、数字等所有字符</li>
            <li>中文字数：统计中文汉字数量</li>
            <li>英文字母：统计英文字母数量</li>
            <li>其他统计：数字、标点、空格、行数等</li>
          </ul>
        </li>
        <li>可以通过开关控制是否将空格计入总字数</li>
        <li>展开详细信息查看更多统计数据</li>
      </ol>
      <div class="note">
        <el-icon><InfoFilled /></el-icon>
        <span>提示：工具支持实时统计，输入文本后会立即显示结果</span>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive } from "vue";
import { useRouter } from "vue-router";
import {
  ArrowLeft,
  Document,
  Delete,
  InfoFilled,
} from "@element-plus/icons-vue";
import { ElMessage } from "element-plus";
import toolsData from "~/data/data.json";

const router = useRouter();
const toolInfo = toolsData.tools.find((tool) => tool.id === 36);

// 更新页面标题和描述
const title = toolInfo?.title || "文本统计";
const description = toolInfo?.description || "文本字数统计工具";

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

// 文本输入
const textInput = ref("");

// 统计设置
const settings = reactive({
  excludeWhitespace: false,
});

// 统计结果类型定义
interface TextStats {
  totalChars: number;
  chineseChars: number;
  englishWords: number;
  numbers: number;
  punctuation: number;
  spaces: number;
  lines: number;
  paragraphs: number;
  uniqueChars: number;
  avgWordLength: number;
  maxLineLength: number;
  textDensity: number;
}

// 统计结果
const stats = reactive<TextStats>({
  totalChars: 0,
  chineseChars: 0,
  englishWords: 0,
  numbers: 0,
  punctuation: 0,
  spaces: 0,
  lines: 0,
  paragraphs: 0,
  uniqueChars: 0,
  avgWordLength: 0,
  maxLineLength: 0,
  textDensity: 0,
});

// 更新统计
const updateStats = () => {
  const text = textInput.value;

  // 重置统计
  Object.keys(stats).forEach((key) => {
    stats[key as keyof TextStats] = 0;
  });

  if (!text) return;

  // 计算基本统计
  stats.totalChars = settings.excludeWhitespace
    ? text.replace(/\s/g, "").length
    : text.length;
  stats.chineseChars = (text.match(/[\u4e00-\u9fa5]/g) || []).length;
  stats.englishWords = (text.match(/[a-zA-Z]/g) || []).length;
  stats.numbers = text.match(/\d/g)?.length || 0;
  stats.punctuation =
    text.match(
      /[!"#$%&'()*+,-./:;<=>?@[\]^_`{|}~。，、；：？！…—·ˉ¨''""々～‖∶＂＇｀｜〃〔〕〈〉《》「」『』．〖〗【】（）［］｛｝]/g
    )?.length || 0;
  stats.spaces = text.match(/\s/g)?.length || 0;
  stats.lines = text.split(/\n/).length;
  stats.paragraphs = text.split(/\n\s*\n/).filter((p) => p.trim()).length;

  // 计算详细统计
  stats.uniqueChars = new Set(text.replace(/\s/g, "")).size;
  const words = text.match(/[a-zA-Z]+/g) || [];
  stats.avgWordLength = words.length
    ? words.reduce((sum, word) => sum + word.length, 0) / words.length
    : 0;
  stats.maxLineLength = Math.max(
    ...text.split("\n").map((line) => line.length)
  );
  const nonWhitespaceChars = text.replace(/\s/g, "").length;
  stats.textDensity = text.length
    ? (nonWhitespaceChars / text.length) * 100
    : 0;
};

// 防抖定时器
let updateTimer: NodeJS.Timeout | null = null;

// 清空输入
const clearInput = () => {
  textInput.value = "";
  updateStats();
};

// 监听输入变化
const handleInput = () => {
  if (updateTimer) {
    clearTimeout(updateTimer);
  }
  updateTimer = setTimeout(() => {
    updateStats();
  }, 300);
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

.editor-section {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 24px;
}

.editor-container {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.editor-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.editor-header h3 {
  font-size: 16px;
  margin: 0;
  color: var(--el-text-color-primary);
}

.editor-actions {
  display: flex;
  gap: 8px;
}

.stats-container {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.stats-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.stats-header h3 {
  font-size: 16px;
  margin: 0;
  color: var(--el-text-color-primary);
}

.stats-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 16px;
}

.stat-item {
  background: var(--el-bg-color-page);
  padding: 16px;
  border-radius: 8px;
  text-align: center;
}

.stat-label {
  color: var(--el-text-color-secondary);
  font-size: 14px;
  margin-bottom: 8px;
}

.stat-value {
  color: var(--el-color-primary);
  font-size: 24px;
  font-weight: 500;
}

.detailed-stats {
  margin-top: 16px;
}

.stat-details {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.detail-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 8px 0;
  border-bottom: 1px solid var(--el-border-color-lighter);
}

.detail-item:last-child {
  border-bottom: none;
}

.detail-label {
  color: var(--el-text-color-regular);
}

.detail-value {
  color: var(--el-text-color-primary);
  font-weight: 500;
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

  .editor-section {
    grid-template-columns: 1fr;
  }

  .stats-grid {
    grid-template-columns: repeat(2, 1fr);
  }
}

@media (max-width: 480px) {
  .stats-grid {
    grid-template-columns: 1fr;
  }
}
</style>
