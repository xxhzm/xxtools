<template>
  <div class="container">
    <div class="tool-header">
      <h1>在线Ping工具</h1>
      <p class="description">检测网站或IP的连通性和响应时间</p>
    </div>

    <el-card class="tool-card">
      <div class="input-area">
        <el-input
          v-model="url"
          placeholder="请输入网址或IP地址，如: www.example.com"
          :prefix-icon="Monitor"
          clearable
          @keyup.enter="handlePing"
        >
          <template #append>
            <el-button @click="handlePing" :loading="loading" type="primary">
              <el-icon v-if="!loading"><Connection /></el-icon>
              开始检测
            </el-button>
          </template>
        </el-input>
      </div>

      <!-- 结果展示 -->
      <div v-if="result" class="result-area">
        <el-descriptions :column="1" border>
          <el-descriptions-item label="检测地址">
            {{ result.url }}
          </el-descriptions-item>
          <el-descriptions-item label="IP地址">
            {{ result.ip }}
          </el-descriptions-item>
          <el-descriptions-item label="服务器位置">
            {{ result.server }}
          </el-descriptions-item>
          <el-descriptions-item label="响应时间">
            <el-tag :type="getTimeType(result.time)" size="small">
              {{ result.time }}
            </el-tag>
          </el-descriptions-item>
        </el-descriptions>
      </div>

      <!-- 使用说明 -->
      <div class="tips">
        <h3>使用说明：</h3>
        <ol>
          <li>输入要检测的网址或IP地址</li>
          <li>点击"开始检测"或按回车键开始检测</li>
          <li>等待结果显示，包含IP地址、服务器位置和响应时间</li>
        </ol>
        <p>注意：响应时间仅供参考，实际访问速度可能会因网络环境而异</p>
      </div>
    </el-card>
  </div>
</template>

<script setup lang="ts">
import { ref } from "vue";
import { Monitor, Connection } from "@element-plus/icons-vue";
import { ElMessage } from "element-plus";

interface PingResult {
  ip: string;
  server: string;
  time: string;
  url: string;
}

const url = ref("");
const loading = ref(false);
const result = ref<PingResult | null>(null);

const handlePing = async () => {
  if (!url.value) {
    ElMessage.warning("请输入要检测的地址");
    return;
  }

  loading.value = true;
  try {
    const response = await fetch(
      `https://v2.xxapi.cn/api/ping?url=${encodeURIComponent(url.value)}`
    );
    const data = await response.json();

    if (data.code === 200) {
      result.value = data.data;
    } else {
      ElMessage.error(data.msg || "检测失败");
    }
  } catch (error) {
    ElMessage.error("网络请求失败，请稍后重试");
  } finally {
    loading.value = false;
  }
};

// 根据响应时间返回不同的标签类型
const getTimeType = (
  time: string
): "success" | "warning" | "danger" | "info" => {
  const ms = parseFloat(time);
  if (ms < 100) return "success";
  if (ms < 200) return "warning";
  return "danger";
};
</script>

<style scoped>
.container {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
}

.tool-header {
  text-align: center;
  margin-bottom: 30px;
}

.tool-header h1 {
  font-size: 24px;
  font-weight: bold;
  margin-bottom: 10px;
}

.description {
  color: #666;
  font-size: 14px;
}

.tool-card {
  margin-bottom: 20px;
}

.input-area {
  margin-bottom: 20px;
}

.result-area {
  margin: 20px 0;
}

.tips {
  margin-top: 30px;
  padding: 15px;
  background-color: var(--el-color-primary-light-9);
  border-radius: 4px;
}

.tips h3 {
  font-size: 16px;
  font-weight: 600;
  margin-bottom: 10px;
}

.tips ol {
  padding-left: 20px;
  margin: 10px 0;
}

.tips li {
  margin-bottom: 5px;
  color: #666;
}

.tips p {
  margin-top: 10px;
  color: #666;
  font-size: 14px;
}

:deep(.el-descriptions__label) {
  width: 120px;
  justify-content: center;
}
</style>
