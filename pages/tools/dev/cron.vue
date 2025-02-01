<template>
  <div class="container">
    <div class="nav-header">
      <el-button link @click="router.push('/')" class="back-btn">
        <el-icon><ArrowLeft /></el-icon>
        返回首页
      </el-button>
    </div>

    <div class="tool-header">
      <el-icon class="header-icon"><Clock /></el-icon>
      <div class="header-text">
        <h1>{{ title }}</h1>
        <p class="subtitle">{{ description }}</p>
      </div>
    </div>

    <div class="tool-content">
      <div class="cron-section">
        <div class="input-section">
          <el-form :model="form" label-width="100px">
            <el-form-item label="Cron表达式">
              <div class="cron-input">
                <el-input
                  v-model="form.expression"
                  placeholder="* * * * * ?"
                  @input="validateExpression"
                >
                  <template #append>
                    <el-button @click="clearExpression">
                      <el-icon><Delete /></el-icon>
                    </el-button>
                  </template>
                </el-input>
              </div>
            </el-form-item>

            <el-tabs v-model="activeTab" class="cron-tabs">
              <el-tab-pane label="秒" name="second">
                <div class="tab-content">
                  <el-radio-group
                    v-model="form.second.type"
                    @change="generateExpression"
                  >
                    <el-radio label="every">每秒</el-radio>
                    <el-radio label="range">周期</el-radio>
                    <el-radio label="steps">步长</el-radio>
                    <el-radio label="specific">指定</el-radio>
                  </el-radio-group>

                  <div class="options" v-if="form.second.type === 'range'">
                    从
                    <el-input-number
                      v-model="form.second.start"
                      :min="0"
                      :max="59"
                      @change="generateExpression"
                    />
                    到
                    <el-input-number
                      v-model="form.second.end"
                      :min="0"
                      :max="59"
                      @change="generateExpression"
                    />
                  </div>

                  <div class="options" v-if="form.second.type === 'steps'">
                    从
                    <el-input-number
                      v-model="form.second.start"
                      :min="0"
                      :max="59"
                      @change="generateExpression"
                    />
                    开始，每
                    <el-input-number
                      v-model="form.second.step"
                      :min="1"
                      :max="59"
                      @change="generateExpression"
                    />
                    秒执行一次
                  </div>

                  <div class="options" v-if="form.second.type === 'specific'">
                    <el-checkbox-group
                      v-model="form.second.specific"
                      @change="generateExpression"
                    >
                      <el-checkbox
                        v-for="i in 60"
                        :key="i - 1"
                        :label="i - 1"
                        >{{ i - 1 }}</el-checkbox
                      >
                    </el-checkbox-group>
                  </div>
                </div>
              </el-tab-pane>

              <el-tab-pane label="分钟" name="minute">
                <div class="tab-content">
                  <el-radio-group
                    v-model="form.minute.type"
                    @change="generateExpression"
                  >
                    <el-radio label="every">每分</el-radio>
                    <el-radio label="range">周期</el-radio>
                    <el-radio label="steps">步长</el-radio>
                    <el-radio label="specific">指定</el-radio>
                  </el-radio-group>

                  <div class="options" v-if="form.minute.type === 'range'">
                    从
                    <el-input-number
                      v-model="form.minute.start"
                      :min="0"
                      :max="59"
                      @change="generateExpression"
                    />
                    到
                    <el-input-number
                      v-model="form.minute.end"
                      :min="0"
                      :max="59"
                      @change="generateExpression"
                    />
                  </div>

                  <div class="options" v-if="form.minute.type === 'steps'">
                    从
                    <el-input-number
                      v-model="form.minute.start"
                      :min="0"
                      :max="59"
                      @change="generateExpression"
                    />
                    开始，每
                    <el-input-number
                      v-model="form.minute.step"
                      :min="1"
                      :max="59"
                      @change="generateExpression"
                    />
                    分钟执行一次
                  </div>

                  <div class="options" v-if="form.minute.type === 'specific'">
                    <el-checkbox-group
                      v-model="form.minute.specific"
                      @change="generateExpression"
                    >
                      <el-checkbox
                        v-for="i in 60"
                        :key="i - 1"
                        :label="i - 1"
                        >{{ i - 1 }}</el-checkbox
                      >
                    </el-checkbox-group>
                  </div>
                </div>
              </el-tab-pane>

              <el-tab-pane label="小时" name="hour">
                <div class="tab-content">
                  <el-radio-group
                    v-model="form.hour.type"
                    @change="generateExpression"
                  >
                    <el-radio label="every">每小时</el-radio>
                    <el-radio label="range">周期</el-radio>
                    <el-radio label="steps">步长</el-radio>
                    <el-radio label="specific">指定</el-radio>
                  </el-radio-group>

                  <div class="options" v-if="form.hour.type === 'range'">
                    从
                    <el-input-number
                      v-model="form.hour.start"
                      :min="0"
                      :max="23"
                      @change="generateExpression"
                    />
                    到
                    <el-input-number
                      v-model="form.hour.end"
                      :min="0"
                      :max="23"
                      @change="generateExpression"
                    />
                  </div>

                  <div class="options" v-if="form.hour.type === 'steps'">
                    从
                    <el-input-number
                      v-model="form.hour.start"
                      :min="0"
                      :max="23"
                      @change="generateExpression"
                    />
                    开始，每
                    <el-input-number
                      v-model="form.hour.step"
                      :min="1"
                      :max="23"
                      @change="generateExpression"
                    />
                    小时执行一次
                  </div>

                  <div class="options" v-if="form.hour.type === 'specific'">
                    <el-checkbox-group
                      v-model="form.hour.specific"
                      @change="generateExpression"
                    >
                      <el-checkbox
                        v-for="i in 24"
                        :key="i - 1"
                        :label="i - 1"
                        >{{ i - 1 }}</el-checkbox
                      >
                    </el-checkbox-group>
                  </div>
                </div>
              </el-tab-pane>

              <el-tab-pane label="日期" name="day">
                <div class="tab-content">
                  <el-radio-group
                    v-model="form.day.type"
                    @change="generateExpression"
                  >
                    <el-radio label="every">每日</el-radio>
                    <el-radio label="range">周期</el-radio>
                    <el-radio label="steps">步长</el-radio>
                    <el-radio label="specific">指定</el-radio>
                    <el-radio label="lastDay">本月最后一天</el-radio>
                    <el-radio label="lastWeekday">本月最后一个工作日</el-radio>
                  </el-radio-group>

                  <div class="options" v-if="form.day.type === 'range'">
                    从
                    <el-input-number
                      v-model="form.day.start"
                      :min="1"
                      :max="31"
                      @change="generateExpression"
                    />
                    到
                    <el-input-number
                      v-model="form.day.end"
                      :min="1"
                      :max="31"
                      @change="generateExpression"
                    />
                  </div>

                  <div class="options" v-if="form.day.type === 'steps'">
                    从
                    <el-input-number
                      v-model="form.day.start"
                      :min="1"
                      :max="31"
                      @change="generateExpression"
                    />
                    开始，每
                    <el-input-number
                      v-model="form.day.step"
                      :min="1"
                      :max="31"
                      @change="generateExpression"
                    />
                    天执行一次
                  </div>

                  <div class="options" v-if="form.day.type === 'specific'">
                    <el-checkbox-group
                      v-model="form.day.specific"
                      @change="generateExpression"
                    >
                      <el-checkbox v-for="i in 31" :key="i" :label="i">{{
                        i
                      }}</el-checkbox>
                    </el-checkbox-group>
                  </div>
                </div>
              </el-tab-pane>

              <el-tab-pane label="月份" name="month">
                <div class="tab-content">
                  <el-radio-group
                    v-model="form.month.type"
                    @change="generateExpression"
                  >
                    <el-radio label="every">每月</el-radio>
                    <el-radio label="range">周期</el-radio>
                    <el-radio label="steps">步长</el-radio>
                    <el-radio label="specific">指定</el-radio>
                  </el-radio-group>

                  <div class="options" v-if="form.month.type === 'range'">
                    从
                    <el-input-number
                      v-model="form.month.start"
                      :min="1"
                      :max="12"
                      @change="generateExpression"
                    />
                    到
                    <el-input-number
                      v-model="form.month.end"
                      :min="1"
                      :max="12"
                      @change="generateExpression"
                    />
                  </div>

                  <div class="options" v-if="form.month.type === 'steps'">
                    从
                    <el-input-number
                      v-model="form.month.start"
                      :min="1"
                      :max="12"
                      @change="generateExpression"
                    />
                    开始，每
                    <el-input-number
                      v-model="form.month.step"
                      :min="1"
                      :max="12"
                      @change="generateExpression"
                    />
                    月执行一次
                  </div>

                  <div class="options" v-if="form.month.type === 'specific'">
                    <el-checkbox-group
                      v-model="form.month.specific"
                      @change="generateExpression"
                    >
                      <el-checkbox v-for="i in 12" :key="i" :label="i"
                        >{{ i }}月</el-checkbox
                      >
                    </el-checkbox-group>
                  </div>
                </div>
              </el-tab-pane>

              <el-tab-pane label="星期" name="week">
                <div class="tab-content">
                  <el-radio-group
                    v-model="form.week.type"
                    @change="generateExpression"
                  >
                    <el-radio label="every">每周</el-radio>
                    <el-radio label="range">周期</el-radio>
                    <el-radio label="steps">步长</el-radio>
                    <el-radio label="specific">指定</el-radio>
                    <el-radio label="lastWeek">本月最后一个星期几</el-radio>
                  </el-radio-group>

                  <div class="options" v-if="form.week.type === 'range'">
                    从
                    <el-select
                      v-model="form.week.start"
                      @change="generateExpression"
                    >
                      <el-option
                        v-for="(day, index) in weekDays"
                        :key="index"
                        :label="day"
                        :value="index"
                      />
                    </el-select>
                    到
                    <el-select
                      v-model="form.week.end"
                      @change="generateExpression"
                    >
                      <el-option
                        v-for="(day, index) in weekDays"
                        :key="index"
                        :label="day"
                        :value="index"
                      />
                    </el-select>
                  </div>

                  <div class="options" v-if="form.week.type === 'steps'">
                    从
                    <el-select
                      v-model="form.week.start"
                      @change="generateExpression"
                    >
                      <el-option
                        v-for="(day, index) in weekDays"
                        :key="index"
                        :label="day"
                        :value="index"
                      />
                    </el-select>
                    开始，每
                    <el-input-number
                      v-model="form.week.step"
                      :min="1"
                      :max="7"
                      @change="generateExpression"
                    />
                    天执行一次
                  </div>

                  <div class="options" v-if="form.week.type === 'specific'">
                    <el-checkbox-group
                      v-model="form.week.specific"
                      @change="generateExpression"
                    >
                      <el-checkbox
                        v-for="(day, index) in weekDays"
                        :key="index"
                        :label="index"
                        >{{ day }}</el-checkbox
                      >
                    </el-checkbox-group>
                  </div>

                  <div class="options" v-if="form.week.type === 'lastWeek'">
                    <el-select
                      v-model="form.week.last"
                      @change="generateExpression"
                    >
                      <el-option
                        v-for="(day, index) in weekDays"
                        :key="index"
                        :label="day"
                        :value="index"
                      />
                    </el-select>
                  </div>
                </div>
              </el-tab-pane>
            </el-tabs>
          </el-form>
        </div>

        <div class="result-section">
          <div class="result-header">
            <h3>最近10次执行时间</h3>
          </div>
          <div class="next-times">
            <ClientOnly>
              <div
                v-for="(time, index) in nextTimes"
                :key="index"
                class="time-item"
              >
                {{ formatDateTime(time) }}
              </div>
            </ClientOnly>
          </div>
        </div>
      </div>
    </div>

    <div class="usage-guide">
      <h2>使用说明</h2>
      <ol>
        <li>
          Cron表达式格式：
          <ul>
            <li>秒 分 时 日 月 星期</li>
            <li>
              每个字段都可以包含特殊字符：* (所有值)、- (范围)、, (列表)、/
              (步长)
            </li>
          </ul>
        </li>
        <li>
          常用示例：
          <ul>
            <li>每分钟执行：0 * * * * ?</li>
            <li>每小时执行：0 0 * * * ?</li>
            <li>每天0点执行：0 0 0 * * ?</li>
            <li>每周一0点执行：0 0 0 ? * MON</li>
            <li>每月1号0点执行：0 0 0 1 * ?</li>
          </ul>
        </li>
        <li>可以通过可视化配置或直接输入表达式</li>
        <li>工具会自动计算最近10次执行时间</li>
      </ol>
      <div class="note">
        <el-icon><InfoFilled /></el-icon>
        <span>注意：? 字符用在日期和星期字段，表示不指定具体值</span>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive } from "vue";
import { useRouter } from "vue-router";
import { ArrowLeft, Clock, Delete, InfoFilled } from "@element-plus/icons-vue";
import toolsData from "~/data/data.json";
import dayjs from "dayjs";
import parser from "cron-parser";

const router = useRouter();
const toolInfo = toolsData.tools.find((tool) => tool.id === 38);

// 更新页面标题和描述
const title = toolInfo?.title || "Cron表达式";
const description = toolInfo?.description || "Cron表达式生成器和解析工具";

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

const activeTab = ref("second");
const weekDays = ["周日", "周一", "周二", "周三", "周四", "周五", "周六"];
const nextTimes = ref<Date[]>([]);

// 表单数据
const form = reactive({
  expression: "* * * * * ?",
  second: {
    type: "every",
    start: 0,
    end: 59,
    step: 1,
    specific: [],
  },
  minute: {
    type: "every",
    start: 0,
    end: 59,
    step: 1,
    specific: [],
  },
  hour: {
    type: "every",
    start: 0,
    end: 23,
    step: 1,
    specific: [],
  },
  day: {
    type: "every",
    start: 1,
    end: 31,
    step: 1,
    specific: [],
  },
  month: {
    type: "every",
    start: 1,
    end: 12,
    step: 1,
    specific: [],
  },
  week: {
    type: "every",
    start: 0,
    end: 6,
    step: 1,
    specific: [],
    last: 0,
  },
});

// 生成表达式
const generateExpression = () => {
  const parts = ["", "", "", "", "", ""];

  // 秒
  parts[0] = generatePart(form.second, 0, 59);
  // 分
  parts[1] = generatePart(form.minute, 0, 59);
  // 时
  parts[2] = generatePart(form.hour, 0, 23);
  // 日
  parts[3] = generateDayPart();
  // 月
  parts[4] = generatePart(form.month, 1, 12);
  // 星期
  parts[5] = generateWeekPart();

  form.expression = parts.join(" ");
  validateExpression();
};

// 生成各部分表达式
const generatePart = (
  part: {
    type: string;
    start: number;
    end: number;
    step: number;
    specific: number[];
  },
  min: number,
  max: number
) => {
  switch (part.type) {
    case "every":
      return "*";
    case "range":
      return `${part.start}-${part.end}`;
    case "steps":
      return `${part.start}/${part.step}`;
    case "specific":
      return part.specific.sort((a, b) => a - b).join(",") || "*";
    default:
      return "*";
  }
};

// 生成日期部分表达式
const generateDayPart = () => {
  switch (form.day.type) {
    case "lastDay":
      return "L";
    case "lastWeekday":
      return "LW";
    default:
      return generatePart(form.day, 1, 31);
  }
};

// 生成星期部分表达式
const generateWeekPart = () => {
  switch (form.week.type) {
    case "lastWeek":
      return `${form.week.last}L`;
    default:
      return form.day.type === "every" ? "?" : generatePart(form.week, 0, 6);
  }
};

// 验证表达式
const validateExpression = () => {
  try {
    // 尝试解析表达式
    parser.parseExpression(form.expression);
    calculateNextTimes();
  } catch (error) {
    console.error("Invalid cron expression:", error);
    nextTimes.value = [];
  }
};

// 计算下次执行时间
const calculateNextTimes = () => {
  try {
    const interval = parser.parseExpression(form.expression);
    nextTimes.value = [];

    // 获取接下来10次执行时间
    for (let i = 0; i < 10; i++) {
      nextTimes.value.push(interval.next().toDate());
    }
  } catch (error) {
    console.error("Error calculating next execution times:", error);
    nextTimes.value = [];
  }
};

// 格式化日期时间
const formatDateTime = (date: Date) => {
  return dayjs(date).format("YYYY-MM-DD HH:mm:ss");
};

// 清空表达式
const clearExpression = () => {
  form.expression = "* * * * * ?";
  validateExpression();
};

// 初始化
generateExpression();
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

.cron-section {
  display: grid;
  grid-template-columns: 2fr 1fr;
  gap: 24px;
}

.input-section {
  background: var(--el-bg-color-page);
  border-radius: 8px;
  padding: 20px;
}

.cron-input {
  display: flex;
  gap: 8px;
  width: 100%;
  margin-bottom: 16px;
}

.cron-input :deep(.el-input) {
  width: 100%;
}

.cron-input :deep(.el-input__wrapper) {
  width: 100%;
}

.cron-input :deep(.el-input-group__append) {
  padding: 0;
}

.cron-tabs {
  margin-top: 20px;
}

.tab-content {
  padding: 16px 0;
}

.options {
  margin-top: 16px;
  padding: 16px;
  background: var(--el-bg-color);
  border-radius: 8px;
  border: 1px solid var(--el-border-color);
}

.result-section {
  background: var(--el-bg-color-page);
  border-radius: 8px;
  padding: 20px;
}

.result-header {
  margin-bottom: 16px;
}

.result-header h3 {
  font-size: 16px;
  margin: 0;
  color: var(--el-text-color-primary);
}

.next-times {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.time-item {
  padding: 12px;
  background: var(--el-bg-color);
  border-radius: 8px;
  border: 1px solid var(--el-border-color);
  color: var(--el-text-color-regular);
  font-size: 14px;
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

:deep(.el-checkbox-group) {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(80px, 1fr));
  gap: 8px;
}

:deep(.el-radio-group) {
  display: flex;
  flex-wrap: wrap;
  gap: 16px;
}

@media (max-width: 768px) {
  .container {
    padding: 15px;
    width: 100%;
    overflow-x: hidden;
  }

  .cron-section {
    grid-template-columns: 1fr;
  }

  .input-section {
    padding: 15px;
  }

  .cron-input {
    width: 100%;
    margin-bottom: 16px;
  }

  .cron-input :deep(.el-input) {
    width: 100%;
  }

  .cron-input :deep(.el-input__wrapper) {
    width: 100%;
  }

  .cron-input :deep(.el-input-group__append) {
    padding: 0;
  }

  .cron-input :deep(.el-input-group__append .el-button) {
    padding: 8px;
  }

  :deep(.el-form) {
    width: 100%;
  }

  :deep(.el-form-item) {
    width: 100%;
    margin-bottom: 16px;
  }

  :deep(.el-form-item__content) {
    width: 100%;
    flex-wrap: wrap;
  }

  :deep(.el-tabs__header) {
    margin: 0 0 16px;
  }

  :deep(.el-tabs__nav-wrap) {
    padding: 0;
  }

  :deep(.el-tabs__nav) {
    width: 100%;
    display: flex;
  }

  :deep(.el-tabs__item) {
    flex: 1;
    text-align: center;
    padding: 0 4px;
    font-size: 14px;
    height: 36px;
    line-height: 36px;
  }

  :deep(.el-radio-group) {
    flex-direction: column;
    gap: 8px;
  }

  :deep(.el-form-item__label) {
    text-align: left;
    margin-bottom: 8px;
    padding: 0;
  }

  .time-item {
    word-break: break-all;
  }

  .usage-guide {
    padding: 16px;
  }

  .usage-guide ol {
    padding-left: 16px;
  }

  .usage-guide li {
    margin-bottom: 8px;
  }
}
</style>
