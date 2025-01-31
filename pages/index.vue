<template>
  <div class="container">
    <!-- 标题部分 -->
    <div class="hero-section">
      <div class="hero-bg"></div>
      <div class="hero-content">
        <div class="hero-left">
          <h1>小小工具箱</h1>
          <p class="subtitle">简单、高效、实用的在线工具集合</p>
          <div class="hero-features">
            <div class="feature-item">
              <el-icon><Check /></el-icon>
              <span>完全免费</span>
            </div>
            <div class="feature-item">
              <el-icon><Connection /></el-icon>
              <span>快速响应</span>
            </div>
            <div class="feature-item">
              <el-icon><Lock /></el-icon>
              <span>安全可靠</span>
            </div>
          </div>
          <div class="search-wrapper">
            <el-input
              v-model="searchQuery"
              placeholder="搜索工具..."
              :prefix-icon="Search"
              clearable
              @input="handleSearch"
              class="search-input"
            >
              <template #append>
                <el-button type="primary" class="search-button">
                  <el-icon><Search /></el-icon>
                  <span>搜索</span>
                </el-button>
              </template>
            </el-input>
          </div>
        </div>
        <div class="hero-right">
          <div class="tools-preview">
            <div class="preview-card">
              <el-icon><Monitor /></el-icon>
            </div>
            <div class="preview-card">
              <el-icon><Document /></el-icon>
            </div>
            <div class="preview-card">
              <el-icon><Link /></el-icon>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- 工具列表 -->
    <div class="tools-list">
      <!-- 按标签分组显示 -->
      <template v-if="filteredTools.length > 0">
        <div v-for="(tools, tag) in groupedTools" :key="tag" class="tool-group">
          <div class="group-header">
            <el-icon class="group-icon"><FolderOpened /></el-icon>
            <h2 class="group-title">{{ tag }}</h2>
          </div>

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
                  <div class="tool-icon-wrapper">
                    <el-icon class="tool-icon"
                      ><component :is="iconMap[tool.icon]"
                    /></el-icon>
                  </div>
                  <div class="tool-content">
                    <div class="tool-header">
                      <span class="title">{{ tool.title }}</span>
                      <el-tag size="small" effect="plain" class="tool-tag">{{
                        tool.tag
                      }}</el-tag>
                    </div>
                    <p class="description">{{ tool.description }}</p>
                    <div class="tool-footer">
                      <span class="try-now">立即使用</span>
                      <el-icon><ArrowRight /></el-icon>
                    </div>
                  </div>
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

    <!-- 问题反馈 -->
    <div class="feedback-section">
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

    <!-- 页脚 -->
    <footer class="footer">
      <div class="friend-links">
        <span class="label">友情链接：</span>
        <div class="links">
          <a
            v-for="link in toolsData.footer.friendLinks"
            :key="link.url"
            :href="link.url"
            target="_blank"
            class="link"
          >
            {{ link.name }}
          </a>
        </div>
      </div>
      <div class="beian">
        <a href="https://beian.miit.gov.cn/" target="_blank" class="beian-link">
          {{ toolsData.footer.beian.icp }}
        </a>
        <span
          v-if="
            toolsData.footer.beian.gongan.text &&
            toolsData.footer.beian.gongan.code
          "
          class="divider"
          >|</span
        >
        <a
          v-if="
            toolsData.footer.beian.gongan.text &&
            toolsData.footer.beian.gongan.code
          "
          :href="
            'http://www.beian.gov.cn/portal/registerSystemInfo?recordcode=' +
            toolsData.footer.beian.gongan.code
          "
          target="_blank"
          class="beian-link"
        >
          <img
            src="@/assets/images/gongan.png"
            alt="公安备案图标"
            class="gongan-icon"
          />
          {{ toolsData.footer.beian.gongan.text }}
        </a>
      </div>
      <div class="copyright">
        {{ toolsData.footer.copyright }}
      </div>
    </footer>
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
  Tools,
  FolderOpened,
  Check,
  Lock,
  Cellphone,
} from "@element-plus/icons-vue";
import { useRouter } from "vue-router";
import toolsData from "~/data/data.json";
import { useHead } from "unhead";

const router = useRouter();
const searchQuery = ref("");

useHead({
  title: "小小工具箱 - 免费在线工具集合",
  meta: [
    {
      name: "description",
      content:
        "小小工具箱是一个免费的在线工具箱，提供网络检测、图像处理等实用工具。简单易用，无需下载安装，让您的在线工作更轻松。",
    },
    {
      name: "keywords",
      content:
        "小小工具箱,在线工具箱,免费工具箱,在线工具,免费工具,网络工具,图像工具,在线检测,免费在线工具",
    },
    {
      name: "viewport",
      content:
        "width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=0",
    },
  ],
});

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
  Search,
  Cellphone,
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

.hero-section {
  margin-bottom: 40px;
  background: var(--el-bg-color);
  padding: 40px;
  border-radius: 12px;
  position: relative;
  min-height: 360px;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.05);
  border: 1px solid var(--el-border-color-light);
}

.hero-bg {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-image: radial-gradient(
      circle at 20% 150%,
      var(--el-color-primary-light-9) 0%,
      transparent 50%
    ),
    radial-gradient(
      circle at 80% -50%,
      var(--el-color-primary-light-9) 0%,
      transparent 50%
    );
  opacity: 0.5;
}

.hero-content {
  position: relative;
  z-index: 1;
  display: flex;
  justify-content: space-between;
  gap: 40px;
  max-width: 1200px;
  margin: 0 auto;
}

.hero-left {
  flex: 1;
  max-width: 600px;
}

.hero-left h1 {
  font-size: 42px;
  font-weight: bold;
  margin-bottom: 16px;
  color: var(--el-text-color-primary);
  line-height: 1.2;
}

.subtitle {
  font-size: 18px;
  color: var(--el-text-color-regular);
  margin-bottom: 32px;
}

.hero-features {
  display: flex;
  gap: 16px;
  margin-bottom: 32px;
}

.feature-item {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 15px;
  background: var(--el-bg-color);
  padding: 8px 16px;
  border-radius: 8px;
  border: 1px solid var(--el-border-color);
  color: var(--el-text-color-regular);
}

.feature-item .el-icon {
  font-size: 16px;
  color: var(--el-color-primary);
}

.hero-right {
  flex: 1;
  max-width: 400px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.tools-preview {
  position: relative;
  width: 100%;
  height: 300px;
}

.preview-card {
  position: absolute;
  width: 100px;
  height: 100px;
  background: var(--el-bg-color);
  border-radius: 12px;
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.05);
  border: 1px solid var(--el-border-color-light);
  animation: float 6s infinite;
}

.preview-card:nth-child(1) {
  top: 20%;
  left: 10%;
  animation-delay: 0s;
}

.preview-card:nth-child(2) {
  top: 40%;
  left: 50%;
  animation-delay: -2s;
}

.preview-card:nth-child(3) {
  top: 60%;
  left: 20%;
  animation-delay: -4s;
}

.preview-card .el-icon {
  font-size: 32px;
  color: var(--el-color-primary);
}

@keyframes float {
  0%,
  100% {
    transform: translateY(0) rotate(0deg);
  }
  50% {
    transform: translateY(-20px) rotate(5deg);
  }
}

.search-wrapper {
  position: relative;
  z-index: 2;
}

.search-input :deep(.el-input__wrapper) {
  background: var(--el-bg-color);
  border: 1px solid var(--el-border-color);
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.05);
  padding-right: 0;
}

.search-input :deep(.el-input__inner) {
  height: 30px;
  line-height: 30px;
  font-size: 14px;
}

.search-input :deep(.el-input-group__append) {
  padding: 0;
  border: none;
  background: none;
}

.search-button {
  height: 30px;
  line-height: 30px;
  padding: 0 30px;
  font-weight: 500;
  border-radius: 0 4px 4px 0;
  font-size: 14px;
  display: flex;
  align-items: center;
}

.search-button .el-icon {
  margin-right: 4px;
  font-size: 14px;
}

.tool-group {
  margin-bottom: 40px;
  background: #fff;
  padding: 24px;
  border-radius: 12px;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.05);
}

.group-header {
  display: flex;
  align-items: center;
  gap: 12px;
  margin-bottom: 24px;
  padding-bottom: 16px;
  border-bottom: 1px solid var(--el-border-color-light);
}

.group-icon {
  font-size: 24px;
  color: var(--el-color-primary);
  background: var(--el-color-primary-light-9);
  padding: 8px;
  border-radius: 8px;
}

.group-title {
  font-size: 20px;
  font-weight: 600;
  color: var(--el-text-color-primary);
  position: relative;
}

.tool-link {
  display: block;
  text-decoration: none;
  color: inherit;
}

.tool-card {
  border-radius: 8px;
  transition: all 0.3s;
  height: 100%;
  border: 1px solid var(--el-border-color-light);
  background: var(--el-bg-color);
}

.tool-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 8px 16px rgba(0, 0, 0, 0.08);
  border-color: var(--el-color-primary-light-5);
}

.tool-icon-wrapper {
  display: flex;
  justify-content: center;
  margin-bottom: 16px;
  padding-top: 8px;
}

.tool-icon {
  font-size: 28px;
  color: var(--el-color-primary);
  background: var(--el-color-primary-light-9);
  padding: 10px;
  border-radius: 8px;
}

.tool-content {
  padding: 0 16px 16px;
}

.tool-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 12px;
}

.tool-header .title {
  font-size: 16px;
  font-weight: 500;
  color: var(--el-text-color-primary);
}

.tool-tag {
  font-size: 12px;
  border-radius: 4px;
}

.description {
  font-size: 14px;
  color: var(--el-text-color-secondary);
  margin-bottom: 16px;
  line-height: 1.6;
  height: 44px;
  overflow: hidden;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
}

.tool-footer {
  display: flex;
  align-items: center;
  gap: 4px;
  color: var(--el-color-primary);
  font-size: 14px;
  padding-top: 12px;
  border-top: 1px solid var(--el-border-color-lighter);
}

.try-now {
  font-weight: 500;
}

.feedback-section {
  display: flex;
  justify-content: center;
  margin-top: 60px;
  margin-bottom: 20px;
}

.feedback-link {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 12px 24px;
  border-radius: 8px;
  background: var(--el-bg-color);
  color: var(--el-text-color-regular);
  text-decoration: none;
  font-size: 15px;
  border: 1px solid var(--el-border-color);
  transition: all 0.3s;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);
}

.feedback-link:hover {
  color: var(--el-color-primary);
  border-color: var(--el-color-primary);
  background: var(--el-color-primary-light-9);
  transform: translateY(-2px);
}

:deep(.el-card__body) {
  padding: 0;
}

:deep(.el-col) {
  padding: 10px;
}

@media (max-width: 1024px) {
  .hero-right {
    display: none;
  }

  .hero-left {
    max-width: 100%;
  }
}

@media (max-width: 768px) {
  .hero-section {
    padding: 40px 20px;
    min-height: auto;
  }

  .hero-left h1 {
    font-size: 36px;
  }

  .subtitle {
    font-size: 18px;
  }

  .hero-features {
    flex-direction: column;
    gap: 12px;
  }

  .feature-item {
    font-size: 14px;
  }
}

.footer {
  margin-top: 60px;
  padding: 40px 0;
  border-top: 1px solid var(--el-border-color-light);
  text-align: center;
}

.friend-links {
  margin-bottom: 20px;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-wrap: wrap;
  gap: 8px;
}

.friend-links .label {
  color: var(--el-text-color-secondary);
  font-size: 14px;
}

.friend-links .links {
  display: flex;
  gap: 16px;
  flex-wrap: wrap;
}

.friend-links .link {
  color: var(--el-text-color-regular);
  text-decoration: none;
  font-size: 14px;
  transition: color 0.3s;
}

.friend-links .link:hover {
  color: var(--el-color-primary);
}

.beian {
  margin-bottom: 12px;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 12px;
  flex-wrap: wrap;
}

.beian-link {
  color: var(--el-text-color-secondary);
  text-decoration: none;
  font-size: 14px;
  display: flex;
  align-items: center;
  gap: 4px;
}

.beian-link:hover {
  color: var(--el-text-color-regular);
}

.gongan-icon {
  width: 16px;
  height: 16px;
}

.divider {
  color: var(--el-text-color-secondary);
}

.copyright {
  color: var(--el-text-color-secondary);
  font-size: 14px;
}

@media (max-width: 768px) {
  .footer {
    padding: 20px 0;
  }

  .friend-links {
    flex-direction: column;
  }

  .beian {
    flex-direction: column;
    gap: 8px;
  }

  .divider {
    display: none;
  }
}
</style>
