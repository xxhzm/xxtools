<template>
  <div class="container">
    <!-- 标题部分 -->
    <h1 class="page-title">小小工具箱</h1>

    <!-- 搜索框 -->
    <div class="search-box">
      <el-input
        v-model="searchQuery"
        placeholder="搜索工具..."
        :prefix-icon="Search"
        clearable
        @input="handleSearch"
      />
    </div>

    <!-- 工具列表 -->
    <div class="tools-list">
      <div class="list-header">
        <h2>常用工具</h2>
        <a
          href="https://jq.qq.com/?_wv=1027&k=92BF8Ndr"
          target="_blank"
          class="feedback-link"
        >
          <el-icon><ChatLineRound /></el-icon>
          问题反馈
          <el-icon><ArrowRight /></el-icon>
        </a>
      </div>

      <!-- 按标签分组显示 -->
      <template v-if="filteredTools.length > 0">
        <div v-for="(tools, tag) in groupedTools" :key="tag" class="tool-group">
          <h3 class="group-title">{{ tag }}</h3>
          <el-row :gutter="20">
            <el-col
              :xs="24"
              :sm="12"
              :md="8"
              :lg="6"
              v-for="tool in tools"
              :key="tool.id"
            >
              <a
                :href="tool.link"
                class="tool-link"
                @click.prevent="navigateTo(tool.link)"
              >
                <el-card class="tool-card" shadow="hover">
                  <div class="card-header">
                    <span class="title">
                      <el-icon class="tool-icon"
                        ><component :is="iconMap[tool.icon]"
                      /></el-icon>
                      {{ tool.title }}
                    </span>
                    <el-tag size="small" effect="plain">{{ tool.tag }}</el-tag>
                  </div>
                  <div class="description">{{ tool.description }}</div>
                </el-card>
              </a>
            </el-col>
          </el-row>
        </div>
      </template>

      <!-- 无搜索结果提示 -->
      <el-empty
        v-if="filteredTools.length === 0"
        description="没有找到相关工具"
      />
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from "vue";
import {
  Search,
  ArrowRight,
  Picture,
  Link,
  Connection,
  Monitor,
  Document,
  Clock,
  ChatLineRound,
} from "@element-plus/icons-vue";
import { useRouter } from "vue-router";
import toolsData from "~/data/tools.json";

const router = useRouter();
const searchQuery = ref("");

// 定义工具接口
interface Tool {
  id: number;
  title: string;
  description: string;
  tag: string;
  icon: keyof typeof iconMap;
  keywords: string[];
  link: string;
}

// 定义图标映射类型
const iconMap = {
  Picture,
  Link,
  Connection,
  Monitor,
  Document,
  Clock,
} as const;

// 搜索功能
const filteredTools = computed(() => {
  const query = searchQuery.value.toLowerCase().trim();
  if (!query) return toolsData.tools as Tool[];

  return (toolsData.tools as Tool[]).filter((tool) => {
    return (
      tool.title.toLowerCase().includes(query) ||
      tool.description.toLowerCase().includes(query) ||
      tool.tag.toLowerCase().includes(query) ||
      tool.keywords.some((keyword) => keyword.toLowerCase().includes(query))
    );
  });
});

// 按标签分组
const groupedTools = computed(() => {
  const groups: { [key: string]: Tool[] } = {};
  filteredTools.value.forEach((tool) => {
    if (!groups[tool.tag]) {
      groups[tool.tag] = [];
    }
    groups[tool.tag].push(tool);
  });
  return groups;
});

// 搜索处理函数
const handleSearch = () => {
  // 可以在这里添加额外的搜索逻辑
  console.log("搜索关键词:", searchQuery.value);
};

// 跳转函数
const navigateTo = (link: string) => {
  router.push(link);
};
</script>

<style scoped>
.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
}

.page-title {
  font-size: 24px;
  font-weight: bold;
  margin-bottom: 20px;
}

.search-box {
  margin-bottom: 30px;
}

.list-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

.list-header h2 {
  font-size: 20px;
  font-weight: 600;
}

.tool-group {
  margin-bottom: 40px;
}

.group-title {
  font-size: 18px;
  font-weight: 600;
  margin-bottom: 16px;
  color: var(--el-text-color-primary);
  position: relative;
  padding-left: 12px;
}

.group-title::before {
  content: "";
  position: absolute;
  left: 0;
  top: 50%;
  transform: translateY(-50%);
  width: 4px;
  height: 16px;
  background-color: var(--el-color-primary);
  border-radius: 2px;
}

.tool-link {
  display: block;
  text-decoration: none;
  color: inherit;
}

.tool-card {
  margin-bottom: 20px;
  height: 100%;
  cursor: pointer;
  transition: all 0.3s;
}

.tool-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  margin-bottom: 12px;
}

.card-header .title {
  font-size: 16px;
  font-weight: 500;
  display: flex;
  align-items: center;
  gap: 8px;
}

.tool-icon {
  font-size: 18px;
  color: var(--el-color-primary);
}

.description {
  font-size: 14px;
  color: #666;
  line-height: 1.5;
}

:deep(.el-card__body) {
  padding: 15px;
}

:deep(.el-col) {
  padding-top: 10px;
  padding-bottom: 10px;
}

.feedback-link {
  display: inline-flex;
  align-items: center;
  gap: 4px;
  padding: 8px 16px;
  border-radius: 20px;
  background: #fff;
  color: var(--el-text-color-regular);
  text-decoration: none;
  font-size: 14px;
  border: 1px solid var(--el-border-color);
  transition: all 0.3s;
}

.feedback-link:hover {
  color: var(--el-color-primary);
  border-color: var(--el-color-primary);
  background: var(--el-color-primary-light-9);
}

.feedback-link .el-icon {
  font-size: 16px;
}
</style>
