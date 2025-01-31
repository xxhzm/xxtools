<template>
  <div class="container">
    <div class="nav-header">
      <el-button link @click="router.push('/')" class="back-btn">
        <el-icon><ArrowLeft /></el-icon>
        返回首页
      </el-button>
    </div>

    <div class="tool-header">
      <el-icon class="header-icon"><Monitor /></el-icon>
      <div class="header-text">
        <h1>端口扫描</h1>
        <p class="subtitle">检测服务器常用端口的开放状态</p>
      </div>
    </div>

    <div class="tool-content">
      <div class="input-section">
        <el-input
          v-model="address"
          placeholder="请输入域名或IP地址，如: xxhzm.cn"
          :prefix-icon="Monitor"
          clearable
          class="address-input"
          @keyup.enter="handleScan"
        />
        <el-button
          type="primary"
          class="scan-btn"
          @click="handleScan"
          :loading="loading"
        >
          开始扫描
        </el-button>
      </div>

      <!-- 结果展示 -->
      <div v-if="scanResult" class="result-section">
        <div class="result-card">
          <el-table :data="portList" stripe>
            <el-table-column prop="port" label="端口" width="180" />
            <el-table-column prop="status" label="状态">
              <template #default="scope">
                <el-tag
                  :type="scope.row.status ? 'success' : 'danger'"
                  size="small"
                >
                  {{ scope.row.status ? "开放" : "关闭" }}
                </el-tag>
              </template>
            </el-table-column>
            <el-table-column prop="service" label="可能的服务" />
          </el-table>
        </div>
      </div>

      <!-- 错误提示 -->
      <el-alert
        v-if="error"
        :title="error"
        type="error"
        :closable="false"
        show-icon
        class="error-alert"
      />
    </div>

    <!-- 使用说明 -->
    <div class="usage-guide">
      <h2>使用说明</h2>
      <ol>
        <li>在输入框中输入需要扫描的域名或IP地址</li>
        <li>点击"开始扫描"按钮或按回车键</li>
        <li>等待扫描完成，查看端口状态</li>
      </ol>
      <div class="note">
        <el-icon><InfoFilled /></el-icon>
        <span
          >注意：扫描结果仅供参考，实际端口状态可能因网络环境或防火墙设置而异</span
        >
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from "vue";
import { Monitor, InfoFilled, ArrowLeft } from "@element-plus/icons-vue";
import { ElMessage } from "element-plus";
import { useHead } from "unhead";
import { useRouter } from "vue-router";
import toolsData from "~/data/data.json";

const router = useRouter();
const address = ref("");
const loading = ref(false);
const error = ref("");
const scanResult = ref<any>(null);

// 从data.json导入工具信息
const toolInfo = toolsData.tools.find((tool) => tool.id === 7);

// 更新页面标题和描述
const title = toolInfo?.title || "端口扫描";
const description = toolInfo?.description || "检测服务器常用端口的开放状态";

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
    {
      name: "viewport",
      content:
        "width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=0",
    },
  ],
});

// 端口服务映射
const portServices: Record<string, string> = {
  "21": "FTP",
  "22": "SSH",
  "80": "HTTP",
  "443": "HTTPS",
  "3306": "MySQL",
  "6379": "Redis",
  "8080": "HTTP Proxy",
  "8888": "HTTP Alternative",
};

// 处理扫描结果
const portList = computed(() => {
  if (!scanResult.value) return [];
  return Object.entries(scanResult.value.data).map(([port, status]) => ({
    port,
    status,
    service: portServices[port] || "未知服务",
  }));
});

// 扫描处理函数
const handleScan = async () => {
  if (!address.value) {
    ElMessage.warning("请输入域名或IP地址");
    return;
  }

  loading.value = true;
  error.value = "";
  scanResult.value = null;

  try {
    const response = await fetch(
      `https://v2.xxapi.cn/api/portscan?address=${encodeURIComponent(
        address.value
      )}`
    );
    const data = await response.json();

    if (data.code === 200) {
      scanResult.value = data;
      ElMessage.success("扫描完成");
    } else {
      error.value = data.msg || "扫描失败，请稍后重试";
    }
  } catch (err) {
    error.value = "网络错误，请稍后重试";
  } finally {
    loading.value = false;
  }
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
  display: flex;
  gap: 16px;
  flex-wrap: wrap;
}

.address-input {
  flex: 1;
  min-width: 200px;
}

.scan-btn {
  margin-top: 16px;
  width: 100%;
}

.result-section {
  padding-top: 24px;
  border-top: 1px solid var(--el-border-color-light);
}

.result-card {
  background: var(--el-bg-color-page);
  border-radius: 8px;
  padding: 20px;
  border: 1px solid var(--el-border-color);
}

.error-alert {
  margin: 20px 0;
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

:deep(.el-table) {
  border-radius: 8px;
  border: 1px solid var(--el-border-color-lighter);
}

@media (max-width: 768px) {
  .container {
    padding: 15px;
  }

  .header h1 {
    font-size: 20px;
  }

  .input-section {
    flex-direction: column;
  }
}
</style>
