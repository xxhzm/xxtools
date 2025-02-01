<template>
  <div class="container">
    <div class="nav-header">
      <el-button link @click="router.push('/')" class="back-btn">
        <el-icon><ArrowLeft /></el-icon>
        返回首页
      </el-button>
    </div>

    <div class="tool-header">
      <el-icon class="header-icon"><Cellphone /></el-icon>
      <div class="header-text">
        <h1>手机归属地</h1>
        <p class="subtitle">查询手机号码归属地、运营商等信息</p>
      </div>
    </div>

    <div class="tool-content">
      <div class="input-section">
        <el-input
          v-model="phone"
          placeholder="请输入手机号码，如: 131********"
          :prefix-icon="Cellphone"
          maxlength="11"
          clearable
          class="phone-input"
          @input="validatePhone"
        />
        <el-button
          type="primary"
          class="query-btn"
          @click="startQuery"
          :loading="loading"
        >
          开始查询
        </el-button>
      </div>

      <div v-if="result" class="result-section">
        <div class="result-card">
          <div class="result-item">
            <span class="label">手机号码：</span>
            <span class="value">{{ result.phone }}</span>
          </div>
          <div class="result-item">
            <span class="label">归属地：</span>
            <span class="value">{{ result.province }}{{ result.city }}</span>
          </div>
          <div class="result-item">
            <span class="label">运营商：</span>
            <span class="value">{{ result.cardType }}</span>
          </div>
          <div class="result-item">
            <span class="label">区号：</span>
            <span class="value">{{ result.areaZone }}</span>
          </div>
          <div class="result-item">
            <span class="label">邮编：</span>
            <span class="value">{{ result.zipCode }}</span>
          </div>
        </div>
      </div>
    </div>

    <div class="usage-guide">
      <h2>使用说明</h2>
      <ol>
        <li>在输入框中输入需要查询的手机号码</li>
        <li>点击"开始查询"按钮</li>
        <li>查看手机号码的归属地等详细信息</li>
      </ol>
      <div class="note">
        <el-icon><InfoFilled /></el-icon>
        <span>注意：查询结果仅供参考，具体信息请以运营商提供的信息为准</span>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from "vue";
import { ElMessage } from "element-plus";
import { useRouter } from "vue-router";
import { Cellphone, InfoFilled, ArrowLeft } from "@element-plus/icons-vue";
import toolsData from "~/data/data.json";

const router = useRouter();
const phone = ref("");
const result = ref<any>(null);
const loading = ref(false);

// 从data.json导入工具信息
const toolInfo = toolsData.tools.find((tool) => tool.id === 6);

// 更新页面标题和描述
const title = toolInfo?.title || "手机归属地";
const description = toolInfo?.description || "查询手机号码归属地、运营商等信息";

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

const validatePhone = (value: string) => {
  // 只允许输入数字
  phone.value = value.replace(/[^\d]/g, "");
};

const startQuery = async () => {
  if (!phone.value.trim()) {
    ElMessage.warning("请输入需要查询的手机号码");
    return;
  }

  if (!/^1[3-9]\d{9}$/.test(phone.value)) {
    ElMessage.warning("请输入正确的手机号码");
    return;
  }

  loading.value = true;
  try {
    const response = await fetch(
      `https://v2.xxapi.cn/api/phoneAddress?phone=${phone.value}`
    );
    const data = await response.json();

    if (data.code === 200 && data.data) {
      result.value = data.data;
      ElMessage.success("查询成功");
    } else {
      ElMessage.error(data.msg || "查询失败，请稍后重试");
    }
  } catch (error) {
    ElMessage.error("网络错误，请稍后重试");
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

.phone-input {
  flex: 1;
  min-width: 200px;
}

.query-btn {
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

.result-item {
  display: flex;
  margin-bottom: 16px;
  line-height: 1.6;
}

.result-item:last-child {
  margin-bottom: 0;
}

.result-item .label {
  width: 80px;
  color: var(--el-text-color-secondary);
  flex-shrink: 0;
}

.result-item .value {
  color: var(--el-text-color-primary);
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
  .input-section {
    flex-direction: column;
  }
}
</style>
