<template>
  <div class="container">
    <div class="nav-header">
      <el-button link @click="router.push('/')" class="back-btn">
        <el-icon><ArrowLeft /></el-icon>
        返回首页
      </el-button>
    </div>

    <div class="tool-header">
      <el-icon class="header-icon"><Timer /></el-icon>
      <div class="header-text">
        <h1>时间戳</h1>
        <p class="subtitle">Unix时间戳与日期时间互转工具，支持毫秒级转换</p>
      </div>
    </div>

    <div class="tool-content">
      <div class="current-time">
        <div class="time-display">
          <div class="time-item">
            <div class="label">当前时间</div>
            <div class="value">{{ currentDateTime }}</div>
          </div>
          <div class="time-item">
            <div class="label">秒级时间戳</div>
            <div class="value">{{ currentTimestamp }}</div>
          </div>
          <div class="time-item">
            <div class="label">毫秒级时间戳</div>
            <div class="value">{{ currentTimestampMs }}</div>
          </div>
        </div>
        <el-button type="primary" @click="refreshTime">
          <el-icon><Refresh /></el-icon>
          刷新
        </el-button>
      </div>

      <el-divider>时间戳转日期时间</el-divider>

      <div class="conversion-section">
        <div class="input-group">
          <el-input
            v-model="timestampInput"
            placeholder="输入时间戳"
            type="number"
            :prefix-icon="Timer"
            clearable
          >
            <template #append>
              <el-select v-model="timestampUnit" style="width: 95px">
                <el-option label="秒" value="s" />
                <el-option label="毫秒" value="ms" />
              </el-select>
            </template>
          </el-input>
          <div class="result" v-if="timestampResult">
            <div class="result-item">
              <span class="label">日期时间：</span>
              <span class="value">{{ timestampResult }}</span>
              <el-button link type="primary" @click="copyText(timestampResult)">
                复制
              </el-button>
            </div>
          </div>
        </div>
      </div>

      <el-divider>日期时间转时间戳</el-divider>

      <div class="conversion-section">
        <div class="input-group">
          <el-date-picker
            v-model="dateTimeInput"
            type="datetime"
            placeholder="选择日期时间"
            format="YYYY-MM-DD HH:mm:ss"
            value-format="x"
            :shortcuts="shortcuts"
            style="width: 100%"
          />
          <div class="result" v-if="dateTimeInput">
            <div class="result-item">
              <span class="label">秒级时间戳：</span>
              <span class="value">{{ dateTimeInput }}</span>
              <el-button link type="primary" @click="copyText(dateTimeInput)">
                复制
              </el-button>
            </div>
            <div class="result-item">
              <span class="label">毫秒级时间戳：</span>
              <span class="value">{{ dateTimeInput + "000" }}</span>
              <el-button
                link
                type="primary"
                @click="copyText(dateTimeInput + '000')"
              >
                复制
              </el-button>
            </div>
          </div>
        </div>
      </div>
    </div>

    <div class="usage-guide">
      <h2>使用说明</h2>
      <ol>
        <li>顶部显示当前时间及对应的时间戳，可点击刷新按钮更新</li>
        <li>
          时间戳转换：输入时间戳数值，选择单位（秒/毫秒），自动显示对应的日期时间
        </li>
        <li>日期转换：选择或输入日期时间，自动显示对应的秒级和毫秒级时间戳</li>
        <li>点击复制按钮可快速复制转换结果</li>
      </ol>
      <div class="note">
        <el-icon><InfoFilled /></el-icon>
        <span
          >提示：Unix时间戳是从1970年1月1日（UTC/GMT的午夜）开始所经过的秒数</span
        >
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
import { Timer, Refresh, InfoFilled, ArrowLeft } from "@element-plus/icons-vue";
import dayjs from "dayjs";
import toolsData from "~/data/data.json";

const router = useRouter();

// 从data.json导入工具信息
const toolInfo = toolsData.tools.find((tool) => tool.id === 16);

// 更新页面标题和描述
const title = toolInfo?.title || "时间戳";
const description = toolInfo?.description || "Unix时间戳与日期时间互转工具";

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

// 当前时间相关
const now = ref(Date.now());
const currentDateTime = computed(() =>
  dayjs(now.value).format("YYYY-MM-DD HH:mm:ss")
);
const currentTimestamp = computed(() => Math.floor(now.value / 1000));
const currentTimestampMs = computed(() => now.value);

const refreshTime = () => {
  now.value = Date.now();
  ElMessage.success("已更新当前时间");
};

// 时间戳转换相关
const timestampInput = ref("");
const timestampUnit = ref("s");
const timestampResult = computed(() => {
  if (!timestampInput.value) return "";
  const timestamp =
    timestampUnit.value === "s"
      ? Number(timestampInput.value) * 1000
      : Number(timestampInput.value);
  return dayjs(timestamp).format("YYYY-MM-DD HH:mm:ss");
});

// 日期时间转换相关
const dateTimeInput = ref("");

// 快捷选项
const shortcuts = [
  {
    text: "现在",
    value: Date.now(),
  },
  {
    text: "昨天",
    value: () => {
      const date = new Date();
      date.setTime(date.getTime() - 3600 * 1000 * 24);
      return date;
    },
  },
  {
    text: "一周前",
    value: () => {
      const date = new Date();
      date.setTime(date.getTime() - 3600 * 1000 * 24 * 7);
      return date;
    },
  },
];

const copyInput = ref<HTMLInputElement | null>(null);

// 修改复制文本函数
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

// 自动更新当前时间
setInterval(() => {
  now.value = Date.now();
}, 1000);
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

.current-time {
  display: flex;
  align-items: flex-start;
  gap: 16px;
  margin-bottom: 24px;
}

.time-display {
  flex: 1;
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 16px;
}

.time-item {
  background: var(--el-bg-color-page);
  border: 1px solid var(--el-border-color);
  border-radius: 6px;
  padding: 12px;
}

.time-item .label {
  color: var(--el-text-color-secondary);
  font-size: 14px;
  margin-bottom: 4px;
}

.time-item .value {
  color: var(--el-text-color-primary);
  font-family: monospace;
  font-size: 16px;
}

.conversion-section {
  margin: 24px 0;
}

.input-group {
  max-width: 500px;
}

.result {
  margin-top: 16px;
  background: var(--el-bg-color-page);
  border: 1px solid var(--el-border-color);
  border-radius: 6px;
  padding: 12px;
}

.result-item {
  display: flex;
  align-items: center;
  gap: 8px;
  margin-bottom: 8px;
}

.result-item:last-child {
  margin-bottom: 0;
}

.result-item .label {
  color: var(--el-text-color-secondary);
  font-size: 14px;
  white-space: nowrap;
}

.result-item .value {
  color: var(--el-text-color-primary);
  font-family: monospace;
  flex: 1;
  word-break: break-all;
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

  .time-display {
    grid-template-columns: 1fr;
  }
}

.copy-input {
  position: absolute;
  top: -9999px;
  left: -9999px;
  opacity: 0;
  pointer-events: none;
}
</style>
