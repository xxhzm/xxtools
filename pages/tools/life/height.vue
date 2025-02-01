<template>
  <div class="container">
    <div class="nav-header">
      <el-button link @click="router.push('/')" class="back-btn">
        <el-icon><ArrowLeft /></el-icon>
        返回首页
      </el-button>
    </div>

    <div class="tool-header">
      <el-icon class="header-icon"><User /></el-icon>
      <div class="header-text">
        <h1>{{ title }}</h1>
        <p class="subtitle">{{ description }}</p>
      </div>
    </div>

    <div class="tool-content">
      <div class="calculator-form">
        <el-form :model="form" label-width="120px" size="large">
          <el-form-item label="父亲身高(cm)">
            <el-input-number
              v-model="form.fatherHeight"
              :min="140"
              :max="220"
              :precision="1"
              :step="0.5"
              @change="calculateHeight"
            />
          </el-form-item>

          <el-form-item label="母亲身高(cm)">
            <el-input-number
              v-model="form.motherHeight"
              :min="140"
              :max="220"
              :precision="1"
              :step="0.5"
              @change="calculateHeight"
            />
          </el-form-item>

          <el-form-item label="孩子性别">
            <el-radio-group v-model="form.gender" @change="calculateHeight">
              <el-radio label="male">男孩</el-radio>
              <el-radio label="female">女孩</el-radio>
            </el-radio-group>
          </el-form-item>
        </el-form>
      </div>

      <div class="result-section" v-if="heightResult.average > 0">
        <div class="result-header">
          <h3>预测身高范围</h3>
        </div>
        <div class="result-content">
          <div class="total-height">
            <div class="height-icon">
              <el-icon><User /></el-icon>
            </div>
            <div class="height-amount">
              <h2>{{ heightResult.average }}cm</h2>
              <p>预测平均身高</p>
            </div>
          </div>
          <div class="height-details">
            <div class="detail-item">
              <h4>最小预测值</h4>
              <p>{{ heightResult.min }}cm</p>
              <small>遗传身高下限</small>
            </div>
            <div class="detail-item">
              <h4>最大预测值</h4>
              <p>{{ heightResult.max }}cm</p>
              <small>遗传身高上限</small>
            </div>
            <div class="detail-item">
              <h4>遗传身高范围</h4>
              <p>±{{ heightResult.range }}cm</p>
              <small>基于平均身高的浮动范围</small>
            </div>
          </div>
        </div>
      </div>
    </div>

    <div class="usage-guide">
      <h2>使用说明</h2>
      <ol>
        <li>输入父亲的身高（厘米）</li>
        <li>输入母亲的身高（厘米）</li>
        <li>选择孩子的性别</li>
        <li>系统会自动计算预测的成年身高范围</li>
      </ol>
      <div class="note">
        <el-icon><InfoFilled /></el-icon>
        <span
          >提示：预测身高仅供参考，实际身高会受到营养、运动等多种因素影响</span
        >
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, reactive } from "vue";
import { useRouter } from "vue-router";
import { ArrowLeft, User, InfoFilled } from "@element-plus/icons-vue";
import toolsData from "~/data/data.json";

const router = useRouter();
const toolInfo = toolsData.tools.find((tool) => tool.id === 32);

// 更新页面标题和描述
const title = toolInfo?.title || "孩子身高预测";
const description = toolInfo?.description || "预测孩子成年后的身高范围";

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
  fatherHeight: 175,
  motherHeight: 162,
  gender: "male",
});

// 计算结果
const heightResult = reactive({
  min: 0,
  max: 0,
  average: 0,
  range: 5, // 默认浮动范围±5cm
});

// 计算身高
const calculateHeight = () => {
  let predictedHeight = 0;

  if (form.gender === "male") {
    // 男孩身高预测：(父亲身高 + 母亲身高 + 13) ÷ 2
    predictedHeight = (form.fatherHeight + form.motherHeight + 13) / 2;
  } else {
    // 女孩身高预测：(父亲身高 - 13 + 母亲身高) ÷ 2
    predictedHeight = (form.fatherHeight - 13 + form.motherHeight) / 2;
  }

  // 更新结果
  heightResult.average = Math.round(predictedHeight);
  heightResult.min = Math.round(predictedHeight - heightResult.range);
  heightResult.max = Math.round(predictedHeight + heightResult.range);
};

// 初始计算
calculateHeight();
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

.calculator-form {
  max-width: 500px;
  margin: 0 auto;
}

.result-section {
  margin-top: 32px;
  padding-top: 24px;
  border-top: 1px solid var(--el-border-color-light);
}

.result-header {
  margin-bottom: 24px;
  text-align: center;
}

.result-header h3 {
  font-size: 18px;
  color: var(--el-text-color-primary);
  margin: 0;
}

.result-content {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 24px;
}

.total-height {
  display: flex;
  align-items: center;
  gap: 16px;
  padding: 24px;
  background: var(--el-color-primary-light-9);
  border-radius: 12px;
  width: 100%;
  max-width: 300px;
}

.height-icon {
  font-size: 48px;
  color: var(--el-color-primary);
}

.height-amount {
  flex: 1;
}

.height-amount h2 {
  font-size: 32px;
  color: var(--el-color-primary);
  margin: 0;
}

.height-amount p {
  margin: 4px 0 0;
  color: var(--el-text-color-secondary);
  font-size: 14px;
}

.height-details {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 16px;
  width: 100%;
}

.detail-item {
  text-align: center;
  padding: 16px;
  background: var(--el-fill-color-light);
  border-radius: 8px;
}

.detail-item h4 {
  font-size: 14px;
  color: var(--el-text-color-primary);
  margin: 0 0 8px;
}

.detail-item p {
  font-size: 20px;
  color: var(--el-color-primary);
  margin: 0 0 4px;
}

.detail-item small {
  font-size: 12px;
  color: var(--el-text-color-secondary);
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

  .height-details {
    grid-template-columns: 1fr;
  }

  .detail-item {
    padding: 12px;
  }
}
</style>
