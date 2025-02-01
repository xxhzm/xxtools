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
        <h1>日期计算</h1>
        <p class="subtitle">
          在线日期相差计算工具，支持计算两个日期之间的天数、工作日等
        </p>
      </div>
    </div>

    <div class="tool-content">
      <div class="date-inputs">
        <el-form :model="dateForm" label-width="100px">
          <el-form-item label="开始日期">
            <el-date-picker
              v-model="dateForm.startDate"
              type="date"
              placeholder="选择开始日期"
              format="YYYY-MM-DD"
              value-format="YYYY-MM-DD"
              @change="calculateDifference"
            />
          </el-form-item>
          <el-form-item label="结束日期">
            <el-date-picker
              v-model="dateForm.endDate"
              type="date"
              placeholder="选择结束日期"
              format="YYYY-MM-DD"
              value-format="YYYY-MM-DD"
              @change="calculateDifference"
            />
          </el-form-item>
          <el-form-item label="计算选项">
            <el-checkbox-group
              v-model="dateForm.options"
              @change="calculateDifference"
            >
              <el-checkbox label="includeEndDate">包含结束日期</el-checkbox>
              <el-checkbox label="excludeWeekends">排除周末</el-checkbox>
            </el-checkbox-group>
          </el-form-item>
        </el-form>
      </div>

      <div v-if="showResults" class="results-section">
        <h3>计算结果</h3>
        <div class="result-items">
          <div class="result-item">
            <div class="label">总天数：</div>
            <div class="value">{{ results.totalDays }} 天</div>
          </div>
          <div class="result-item">
            <div class="label">工作日：</div>
            <div class="value">{{ results.workDays }} 天</div>
          </div>
          <div class="result-item">
            <div class="label">周末天数：</div>
            <div class="value">{{ results.weekendDays }} 天</div>
          </div>
          <div class="result-item">
            <div class="label">相差时间：</div>
            <div class="value">{{ results.timeDiff }}</div>
          </div>
        </div>
      </div>
    </div>

    <div class="usage-guide">
      <h2>使用说明</h2>
      <ol>
        <li>选择开始日期和结束日期</li>
        <li>
          选择计算选项：
          <ul>
            <li>包含结束日期：计算时包含结束日期当天</li>
            <li>排除周末：不计算周六和周日</li>
          </ul>
        </li>
        <li>系统会自动计算并显示相关的日期差值</li>
      </ol>
      <div class="note">
        <el-icon><InfoFilled /></el-icon>
        <span>提示：工作日计算会自动排除周六和周日</span>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive, computed } from "vue";
import { useRouter } from "vue-router";
import { ArrowLeft, Calendar, InfoFilled } from "@element-plus/icons-vue";
import dayjs from "dayjs";
import toolsData from "~/data/data.json";

const router = useRouter();

// 从data.json导入工具信息
const toolInfo = toolsData.tools.find((tool) => tool.id === 23);

// 更新页面标题和描述
const title = toolInfo?.title || "日期计算";
const description = toolInfo?.description || "在线日期相差计算工具";

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
const dateForm = reactive({
  startDate: "",
  endDate: "",
  options: ["includeEndDate"],
});

// 计算结果
const results = reactive({
  totalDays: 0,
  workDays: 0,
  weekendDays: 0,
  timeDiff: "",
});

// 是否显示结果
const showResults = computed(() => dateForm.startDate && dateForm.endDate);

// 计算日期差值
const calculateDifference = () => {
  if (!dateForm.startDate || !dateForm.endDate) return;

  const start = dayjs(dateForm.startDate);
  const end = dayjs(dateForm.endDate);

  // 如果开始日期大于结束日期，交换它们
  if (start.isAfter(end)) {
    const temp = dateForm.startDate;
    dateForm.startDate = dateForm.endDate;
    dateForm.endDate = temp;
    return calculateDifference();
  }

  // 计算总天数
  let days = end.diff(start, "day");
  if (dateForm.options.includes("includeEndDate")) {
    days += 1;
  }

  // 初始化结果
  results.totalDays = days;
  results.workDays = days;
  results.weekendDays = 0;

  // 计算周末天数
  if (dateForm.options.includes("excludeWeekends")) {
    let currentDate = start;
    let weekendCount = 0;

    for (let i = 0; i < days; i++) {
      const dayOfWeek = currentDate.day();
      if (dayOfWeek === 0 || dayOfWeek === 6) {
        weekendCount++;
      }
      currentDate = currentDate.add(1, "day");
    }

    results.weekendDays = weekendCount;
    results.workDays -= weekendCount;
  }

  // 计算时间差的表述
  const years = Math.floor(days / 365);
  const months = Math.floor((days % 365) / 30);
  const remainingDays = days % 30;

  let timeDiff = "";
  if (years > 0) {
    timeDiff += `${years}年`;
  }
  if (months > 0) {
    timeDiff += `${months}个月`;
  }
  if (remainingDays > 0 || timeDiff === "") {
    timeDiff += `${remainingDays}天`;
  }

  results.timeDiff = timeDiff;
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

.date-inputs {
  margin-bottom: 24px;
}

.results-section {
  border-top: 1px solid var(--el-border-color);
  padding-top: 24px;
}

.results-section h3 {
  font-size: 16px;
  margin-bottom: 16px;
  color: var(--el-text-color-primary);
}

.result-items {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 16px;
}

.result-item {
  display: flex;
  align-items: center;
  gap: 8px;
}

.result-item .label {
  color: var(--el-text-color-regular);
  white-space: nowrap;
}

.result-item .value {
  color: var(--el-color-primary);
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
}

.usage-guide li {
  margin-bottom: 8px;
  line-height: 1.6;
}

.usage-guide ul {
  margin: 8px 0;
  padding-left: 20px;
  color: var(--el-text-color-regular);
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

  .result-items {
    grid-template-columns: 1fr;
  }
}
</style>
