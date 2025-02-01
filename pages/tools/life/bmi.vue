<template>
  <div class="container">
    <div class="nav-header">
      <el-button link @click="router.push('/')" class="back-btn">
        <el-icon><ArrowLeft /></el-icon>
        返回首页
      </el-button>
    </div>

    <div class="tool-header">
      <el-icon class="header-icon"><DataLine /></el-icon>
      <div class="header-text">
        <h1>{{ title }}</h1>
        <p class="subtitle">{{ description }}</p>
      </div>
    </div>

    <div class="tool-content">
      <div class="calculator-form">
        <el-form :model="form" label-width="120px" size="large">
          <el-form-item label="身高(cm)">
            <el-input-number
              v-model="form.height"
              :min="100"
              :max="250"
              :precision="1"
              :step="0.5"
              @change="calculateBMI"
            />
          </el-form-item>

          <el-form-item label="体重(kg)">
            <el-input-number
              v-model="form.weight"
              :min="20"
              :max="200"
              :precision="1"
              :step="0.5"
              @change="calculateBMI"
            />
          </el-form-item>

          <el-form-item label="年龄">
            <el-input-number
              v-model="form.age"
              :min="2"
              :max="120"
              :step="1"
              @change="calculateBMI"
            />
          </el-form-item>

          <el-form-item label="性别">
            <el-radio-group v-model="form.gender" @change="calculateBMI">
              <el-radio label="male">男性</el-radio>
              <el-radio label="female">女性</el-radio>
            </el-radio-group>
          </el-form-item>
        </el-form>
      </div>

      <div class="result-section" v-if="bmiResult.value > 0">
        <div class="result-header">
          <h3>BMI计算结果</h3>
        </div>
        <div class="result-content">
          <div class="total-bmi">
            <div class="bmi-icon">
              <el-icon><DataLine /></el-icon>
            </div>
            <div class="bmi-amount">
              <h2>{{ bmiResult.value.toFixed(1) }}</h2>
              <p :style="{ color: getBMIStatusColor() }">
                {{ bmiResult.status }}
              </p>
            </div>
          </div>
          <div class="bmi-details">
            <div class="detail-item">
              <h4>标准体重范围</h4>
              <p>
                {{ bmiResult.idealWeight.min.toFixed(1) }} -
                {{ bmiResult.idealWeight.max.toFixed(1) }}kg
              </p>
              <small>基于身高计算的理想体重范围</small>
            </div>
            <div class="detail-item">
              <h4>体重差值</h4>
              <p :style="{ color: getWeightDiffColor() }">
                {{ getWeightDiffText() }}
              </p>
              <small>与理想体重的差距</small>
            </div>
            <div class="detail-item">
              <h4>基础代谢</h4>
              <p>{{ bmiResult.bmr.toFixed(0) }}千卡</p>
              <small>每日基础代谢消耗</small>
            </div>
          </div>
        </div>
      </div>
    </div>

    <div class="bmi-scale">
      <h3>BMI参考标准</h3>
      <div class="scale-container">
        <div class="scale-item" style="background: #95cc39">
          <span class="scale-range">＜18.5</span>
          <span class="scale-label">偏瘦</span>
        </div>
        <div class="scale-item" style="background: #87d068">
          <span class="scale-range">18.5-24.9</span>
          <span class="scale-label">正常</span>
        </div>
        <div class="scale-item" style="background: #ffa940">
          <span class="scale-range">25-29.9</span>
          <span class="scale-label">超重</span>
        </div>
        <div class="scale-item" style="background: #ff4d4f">
          <span class="scale-range">≥30</span>
          <span class="scale-label">肥胖</span>
        </div>
      </div>
    </div>

    <div class="usage-guide">
      <h2>使用说明</h2>
      <ol>
        <li>输入您的身高（厘米）</li>
        <li>输入您的体重（公斤）</li>
        <li>输入您的年龄</li>
        <li>选择您的性别</li>
        <li>系统会自动计算BMI值和健康建议</li>
      </ol>
      <div class="note">
        <el-icon><InfoFilled /></el-icon>
        <span
          >提示：BMI仅作为健康评估的参考指标之一，不同年龄、性别和体型的人可能有所差异</span
        >
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, reactive } from "vue";
import { useRouter } from "vue-router";
import { ArrowLeft, DataLine, InfoFilled } from "@element-plus/icons-vue";
import toolsData from "~/data/data.json";

const router = useRouter();
const toolInfo = toolsData.tools.find((tool) => tool.id === 33);

// 更新页面标题和描述
const title = toolInfo?.title || "BMI计算";
const description =
  toolInfo?.description || "计算身体质量指数(BMI)并评估健康状况";

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
  height: 170,
  weight: 65,
  age: 25,
  gender: "male",
});

// 计算结果
const bmiResult = reactive({
  value: 0,
  status: "",
  idealWeight: {
    min: 0,
    max: 0,
  },
  bmr: 0,
});

// 计算BMI
const calculateBMI = () => {
  // 计算BMI：体重(kg) / 身高(m)²
  const heightInMeter = form.height / 100;
  const bmi = form.weight / (heightInMeter * heightInMeter);

  // 计算理想体重范围（BMI 18.5-24.9）
  const minWeight = 18.5 * heightInMeter * heightInMeter;
  const maxWeight = 24.9 * heightInMeter * heightInMeter;

  // 计算基础代谢率(BMR)
  let bmr = 0;
  if (form.gender === "male") {
    // 男性：10 × 体重(kg) + 6.25 × 身高(cm) - 5 × 年龄 + 5
    bmr = 10 * form.weight + 6.25 * form.height - 5 * form.age + 5;
  } else {
    // 女性：10 × 体重(kg) + 6.25 × 身高(cm) - 5 × 年龄 - 161
    bmr = 10 * form.weight + 6.25 * form.height - 5 * form.age - 161;
  }

  // 更新结果
  bmiResult.value = bmi;
  bmiResult.idealWeight.min = minWeight;
  bmiResult.idealWeight.max = maxWeight;
  bmiResult.bmr = bmr;

  // 设置BMI状态
  if (bmi < 18.5) {
    bmiResult.status = "偏瘦";
  } else if (bmi < 25) {
    bmiResult.status = "正常";
  } else if (bmi < 30) {
    bmiResult.status = "超重";
  } else {
    bmiResult.status = "肥胖";
  }
};

// 获取BMI状态对应的颜色
const getBMIStatusColor = () => {
  switch (bmiResult.status) {
    case "偏瘦":
      return "#95cc39";
    case "正常":
      return "#87d068";
    case "超重":
      return "#ffa940";
    case "肥胖":
      return "#ff4d4f";
    default:
      return "var(--el-text-color-primary)";
  }
};

// 获取体重差值文本
const getWeightDiffText = () => {
  const diff =
    form.weight - (bmiResult.idealWeight.min + bmiResult.idealWeight.max) / 2;
  if (Math.abs(diff) < 0.5) return "体重正常";
  return diff > 0
    ? `偏重${diff.toFixed(1)}kg`
    : `偏轻${Math.abs(diff).toFixed(1)}kg`;
};

// 获取体重差值颜色
const getWeightDiffColor = () => {
  const diff =
    form.weight - (bmiResult.idealWeight.min + bmiResult.idealWeight.max) / 2;
  if (Math.abs(diff) < 0.5) return "#87d068";
  return Math.abs(diff) < 5 ? "#ffa940" : "#ff4d4f";
};

// 初始计算
calculateBMI();
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

.total-bmi {
  display: flex;
  align-items: center;
  gap: 16px;
  padding: 24px;
  background: var(--el-color-primary-light-9);
  border-radius: 12px;
  width: 100%;
  max-width: 300px;
}

.bmi-icon {
  font-size: 48px;
  color: var(--el-color-primary);
}

.bmi-amount {
  flex: 1;
}

.bmi-amount h2 {
  font-size: 32px;
  color: var(--el-color-primary);
  margin: 0;
}

.bmi-amount p {
  margin: 4px 0 0;
  font-size: 16px;
  font-weight: 500;
}

.bmi-details {
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

.bmi-scale {
  background: var(--el-bg-color);
  border-radius: 8px;
  padding: 24px;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.05);
  margin-bottom: 24px;
}

.bmi-scale h3 {
  font-size: 18px;
  margin: 0 0 16px;
  color: var(--el-text-color-primary);
}

.scale-container {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 8px;
}

.scale-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 12px;
  border-radius: 6px;
  color: #fff;
}

.scale-range {
  font-size: 14px;
  margin-bottom: 4px;
}

.scale-label {
  font-size: 16px;
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

  .bmi-details {
    grid-template-columns: 1fr;
  }

  .scale-container {
    grid-template-columns: 1fr;
  }

  .detail-item {
    padding: 12px;
  }
}
</style>
