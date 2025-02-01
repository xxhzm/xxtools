<template>
  <div class="container">
    <div class="nav-header">
      <el-button link @click="router.push('/')" class="back-btn">
        <el-icon><ArrowLeft /></el-icon>
        返回首页
      </el-button>
    </div>

    <div class="tool-header">
      <el-icon class="header-icon"><Mug /></el-icon>
      <div class="header-text">
        <h1>{{ title }}</h1>
        <p class="subtitle">{{ description }}</p>
      </div>
    </div>

    <div class="tool-content">
      <div class="calculator-form">
        <el-form :model="form" label-width="120px" size="large">
          <el-form-item label="体重(kg)">
            <el-input-number
              v-model="form.weight"
              :min="30"
              :max="200"
              :precision="1"
              :step="0.5"
              @change="calculateWater"
            />
          </el-form-item>

          <el-form-item label="年龄">
            <el-input-number
              v-model="form.age"
              :min="1"
              :max="120"
              :step="1"
              @change="calculateWater"
            />
          </el-form-item>

          <el-form-item label="每周活动量">
            <el-select v-model="form.activityLevel" @change="calculateWater">
              <el-option
                v-for="item in activityLevels"
                :key="item.value"
                :label="item.label"
                :value="item.value"
              >
                <span>{{ item.label }}</span>
                <span class="activity-desc">{{ item.description }}</span>
              </el-option>
            </el-select>
          </el-form-item>

          <el-form-item label="气候">
            <el-select v-model="form.climate" @change="calculateWater">
              <el-option label="寒冷" value="cold" />
              <el-option label="温和" value="moderate" />
              <el-option label="炎热" value="hot" />
            </el-select>
          </el-form-item>
        </el-form>
      </div>

      <div class="result-section" v-if="waterResult.total > 0">
        <div class="result-header">
          <h3>每日建议饮水量</h3>
        </div>
        <div class="result-content">
          <div class="total-water">
            <div class="water-icon">
              <el-icon><Mug /></el-icon>
            </div>
            <div class="water-amount">
              <h2>{{ waterResult.total }}ml</h2>
              <p>总饮水量</p>
            </div>
          </div>
          <div class="water-details">
            <div class="detail-item">
              <h4>基础饮水量</h4>
              <p>{{ waterResult.base }}ml</p>
              <small>根据体重计算的基础需求</small>
            </div>
            <div class="detail-item">
              <h4>活动调整</h4>
              <p>{{ waterResult.activity }}ml</p>
              <small>根据活动量增加的需求</small>
            </div>
            <div class="detail-item">
              <h4>气候调整</h4>
              <p>{{ waterResult.climate }}ml</p>
              <small>根据气候条件调整的量</small>
            </div>
          </div>
        </div>
      </div>
    </div>

    <div class="usage-guide">
      <h2>使用说明</h2>
      <ol>
        <li>输入您的体重和年龄</li>
        <li>选择您的每周活动量级别</li>
        <li>选择您所在地区的气候类型</li>
        <li>系统会自动计算您的每日建议饮水量</li>
      </ol>
      <div class="note">
        <el-icon><InfoFilled /></el-icon>
        <span>提示：建议饮水量仅供参考，实际饮水需求可能因个人情况而异</span>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, reactive } from "vue";
import { useRouter } from "vue-router";
import { ArrowLeft, Mug, InfoFilled } from "@element-plus/icons-vue";
import toolsData from "~/data/data.json";

const router = useRouter();
const toolInfo = toolsData.tools.find((tool) => tool.id === 31);

// 更新页面标题和描述
const title = toolInfo?.title || "每日喝水";
const description = toolInfo?.description || "计算每日建议饮水量";

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

// 活动量级别
const activityLevels = [
  {
    value: "sedentary",
    label: "久坐",
    description: "几乎没有运动",
    factor: 1.0,
  },
  {
    value: "light",
    label: "轻度活动",
    description: "每周轻度运动1-3天",
    factor: 1.2,
  },
  {
    value: "moderate",
    label: "中度活动",
    description: "每周中度运动3-5天",
    factor: 1.4,
  },
  {
    value: "active",
    label: "高度活动",
    description: "每周高强度运动6-7天",
    factor: 1.6,
  },
  {
    value: "very_active",
    label: "极高度活动",
    description: "每天极高强度运动2次以上",
    factor: 1.8,
  },
];

// 表单数据
const form = reactive({
  weight: 60,
  age: 25,
  activityLevel: "moderate",
  climate: "moderate",
});

// 计算结果
const waterResult = reactive({
  base: 0,
  activity: 0,
  climate: 0,
  total: 0,
});

// 计算饮水量
const calculateWater = () => {
  // 基础饮水量：体重 * 30ml
  const baseWater = form.weight * 30;

  // 活动量调整
  const activityLevel = activityLevels.find(
    (level) => level.value === form.activityLevel
  );
  const activityWater = baseWater * (activityLevel?.factor || 1.0) - baseWater;

  // 气候调整
  let climateWater = 0;
  switch (form.climate) {
    case "hot":
      climateWater = baseWater * 0.2; // 炎热天气增加20%
      break;
    case "cold":
      climateWater = baseWater * -0.1; // 寒冷天气减少10%
      break;
    default:
      climateWater = 0;
  }

  // 年龄调整
  let ageAdjustment = 1;
  if (form.age > 60) {
    ageAdjustment = 0.9; // 老年人略微减少
  } else if (form.age < 18) {
    ageAdjustment = 1.1; // 青少年略微增加
  }

  // 更新结果
  waterResult.base = Math.round(baseWater);
  waterResult.activity = Math.round(activityWater);
  waterResult.climate = Math.round(climateWater);
  waterResult.total = Math.round(
    (baseWater + activityWater + climateWater) * ageAdjustment
  );
};

// 初始计算
calculateWater();
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

.activity-desc {
  margin-left: 8px;
  color: var(--el-text-color-secondary);
  font-size: 12px;
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

.total-water {
  display: flex;
  align-items: center;
  gap: 16px;
  padding: 24px;
  background: var(--el-color-primary-light-9);
  border-radius: 12px;
  width: 100%;
  max-width: 300px;
}

.water-icon {
  font-size: 48px;
  color: var(--el-color-primary);
}

.water-amount {
  flex: 1;
}

.water-amount h2 {
  font-size: 32px;
  color: var(--el-color-primary);
  margin: 0;
}

.water-amount p {
  margin: 4px 0 0;
  color: var(--el-text-color-secondary);
  font-size: 14px;
}

.water-details {
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

  .water-details {
    grid-template-columns: 1fr;
  }

  .detail-item {
    padding: 12px;
  }
}
</style>
