<template>
  <div class="container">
    <div class="nav-header">
      <el-button link @click="router.push('/')" class="back-btn">
        <el-icon><ArrowLeft /></el-icon>
        返回首页
      </el-button>
    </div>

    <div class="tool-header">
      <el-icon class="header-icon"><Calendar /></el-icon>
      <div class="header-text">
        <h1>历史上的今天</h1>
        <p class="subtitle">查看历史上今天发生的重大事件</p>
      </div>
    </div>

    <div class="tool-content">
      <div class="date-section">
        <div class="current-date">
          <el-icon><Calendar /></el-icon>
          <span>{{ currentDate }}</span>
        </div>
        <el-button type="primary" @click="refreshHistory" :loading="loading">
          <el-icon><Refresh /></el-icon>
          刷新
        </el-button>
      </div>

      <div v-if="loading" class="loading-section">
        <el-skeleton :rows="5" animated />
      </div>

      <div v-else-if="historyEvents.length" class="history-list">
        <div
          v-for="(event, index) in historyEvents"
          :key="index"
          class="history-item"
        >
          <div class="event-content">
            <span class="event-text">{{ event }}</span>
            <el-button link type="primary" @click="copyText(event)">
              复制
            </el-button>
          </div>
        </div>
      </div>

      <div v-else-if="error" class="error-message">
        <el-alert :title="error" type="error" show-icon />
      </div>

      <div v-else class="empty-message">
        <el-empty description="暂无历史事件数据" />
      </div>
    </div>

    <div class="usage-guide">
      <h2>使用说明</h2>
      <ol>
        <li>页面自动显示今天的历史事件</li>
        <li>点击刷新按钮可以重新获取数据</li>
        <li>点击复制按钮可以复制单条历史事件</li>
      </ol>
      <div class="note">
        <el-icon><InfoFilled /></el-icon>
        <span>提示：每天的历史事件会定期更新</span>
      </div>
    </div>

    <!-- 添加一个隐藏的input用于复制 -->
    <input ref="copyInput" type="text" class="copy-input" />
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, computed } from "vue";
import { ElMessage } from "element-plus";
import { useRouter } from "vue-router";
import {
  Calendar,
  Refresh,
  InfoFilled,
  ArrowLeft,
} from "@element-plus/icons-vue";
import dayjs from "dayjs";
import toolsData from "~/data/data.json";

const router = useRouter();
const copyInput = ref<HTMLInputElement | null>(null);

// 从data.json导入工具信息
const toolInfo = toolsData.tools.find((tool) => tool.id === 18);

// 更新页面标题和描述
const title = toolInfo?.title || "历史上的今天";
const description = toolInfo?.description || "查看历史上今天发生的重大事件";

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

const loading = ref(false);
const error = ref("");
const historyEvents = ref<string[]>([]);

// 格式化当前日期
const currentDate = computed(() => {
  const now = new Date();
  return dayjs(now).format("YYYY年MM月DD日");
});

// 获取历史上的今天
const fetchHistory = async () => {
  loading.value = true;
  error.value = "";
  historyEvents.value = [];

  try {
    const response = await fetch("https://v2.xxapi.cn/api/history");
    const data = await response.json();

    if (data.code === 200 && data.data) {
      historyEvents.value = data.data;
    } else {
      error.value = data.msg || "获取数据失败";
    }
  } catch (err) {
    error.value = "网络错误，请稍后重试";
  } finally {
    loading.value = false;
  }
};

// 刷新数据
const refreshHistory = () => {
  fetchHistory();
};

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

// 页面加载时获取数据
onMounted(() => {
  fetchHistory();
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

.date-section {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 24px;
}

.current-date {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 16px;
  color: var(--el-text-color-primary);
}

.current-date .el-icon {
  color: var(--el-color-primary);
}

.history-list {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.history-item {
  background: var(--el-bg-color-page);
  border: 1px solid var(--el-border-color);
  border-radius: 8px;
  padding: 16px;
}

.event-content {
  display: flex;
  align-items: flex-start;
  gap: 12px;
}

.event-text {
  flex: 1;
  color: var(--el-text-color-primary);
  line-height: 1.6;
}

.loading-section {
  padding: 20px 0;
}

.error-message,
.empty-message {
  margin-top: 20px;
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
