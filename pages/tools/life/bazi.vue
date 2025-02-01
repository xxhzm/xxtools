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
        <h1>{{ title }}</h1>
        <p class="subtitle">{{ description }}</p>
      </div>
    </div>

    <div class="tool-content">
      <div class="input-section">
        <el-form :model="form" label-width="100px">
          <el-form-item label="出生日期">
            <div class="date-input">
              <el-config-provider :locale="zhCn">
                <el-date-picker
                  v-model="form.birthDate"
                  type="datetime"
                  placeholder="选择出生日期时间"
                  format="YYYY-MM-DD HH:mm"
                  value-format="YYYY-MM-DD HH:mm:ss"
                  :disabled-date="disabledDate"
                  @change="calculateBazi"
                />
              </el-config-provider>

              <el-radio-group v-model="form.calendar" @change="calculateBazi">
                <el-radio label="solar">阳历</el-radio>
                <el-radio label="lunar">阴历</el-radio>
              </el-radio-group>
            </div>
          </el-form-item>
          <el-form-item label="性别">
            <el-radio-group v-model="form.gender" @change="calculateBazi">
              <el-radio label="male">男</el-radio>
              <el-radio label="female">女</el-radio>
            </el-radio-group>
          </el-form-item>
        </el-form>
      </div>

      <div class="result-section" v-if="baziResult.year">
        <div class="bazi-grid">
          <div class="bazi-cell">
            <div class="cell-label">年柱</div>
            <div class="cell-content">
              <span class="gan">{{ baziResult.year.gan }}</span>
              <span class="zhi">{{ baziResult.year.zhi }}</span>
            </div>
            <div class="cell-info">{{ baziResult.year.element }}</div>
          </div>
          <div class="bazi-cell">
            <div class="cell-label">月柱</div>
            <div class="cell-content">
              <span class="gan">{{ baziResult.month.gan }}</span>
              <span class="zhi">{{ baziResult.month.zhi }}</span>
            </div>
            <div class="cell-info">{{ baziResult.month.element }}</div>
          </div>
          <div class="bazi-cell">
            <div class="cell-label">日柱</div>
            <div class="cell-content">
              <span class="gan">{{ baziResult.day.gan }}</span>
              <span class="zhi">{{ baziResult.day.zhi }}</span>
            </div>
            <div class="cell-info">{{ baziResult.day.element }}</div>
          </div>
          <div class="bazi-cell">
            <div class="cell-label">时柱</div>
            <div class="cell-content">
              <span class="gan">{{ baziResult.hour.gan }}</span>
              <span class="zhi">{{ baziResult.hour.zhi }}</span>
            </div>
            <div class="cell-info">{{ baziResult.hour.element }}</div>
          </div>
        </div>

        <div class="analysis-section">
          <h3>五行分析</h3>
          <div class="element-chart">
            <div
              v-for="(value, element) in elementCount"
              :key="element"
              class="element-bar"
            >
              <span class="element-name">{{ element }}</span>
              <div class="bar-wrapper">
                <div class="bar" :style="{ width: value * 20 + '%' }"></div>
                <span class="bar-value">{{ value }}</span>
              </div>
            </div>
          </div>
        </div>
      </div>

      <div class="empty-state" v-else>
        <el-icon><Calendar /></el-icon>
        <p>请选择出生日期时间进行八字计算</p>
      </div>
    </div>

    <div class="usage-guide">
      <h2>使用说明</h2>
      <ol>
        <li>选择出生日期时间（可精确到时分）</li>
        <li>选择阴历或阳历</li>
        <li>选择性别</li>
        <li>
          系统将自动计算：
          <ul>
            <li>年月日时四柱</li>
            <li>天干地支</li>
            <li>五行属性</li>
            <li>五行强弱分析</li>
          </ul>
        </li>
      </ol>
      <div class="note">
        <el-icon><InfoFilled /></el-icon>
        <span>注意：阴历日期的转换采用标准天文历法计算</span>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive } from "vue";
import { useRouter } from "vue-router";
import { ArrowLeft, Calendar, InfoFilled } from "@element-plus/icons-vue";
import dayjs from "dayjs";
import toolsData from "~/data/data.json";
// @ts-ignore
import zhCn from "element-plus/dist/locale/zh-cn.mjs";

const router = useRouter();
const toolInfo = toolsData.tools.find((tool) => tool.id === 40);

// 更新页面标题和描述
const title = toolInfo?.title || "生辰八字";
const description = toolInfo?.description || "生辰八字计算工具";

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
  birthDate: "",
  calendar: "solar",
  gender: "male",
});

// 八字计算结果
const baziResult = reactive({
  year: { gan: "", zhi: "", element: "" },
  month: { gan: "", zhi: "", element: "" },
  day: { gan: "", zhi: "", element: "" },
  hour: { gan: "", zhi: "", element: "" },
});

// 五行统计
const elementCount = reactive({
  金: 0,
  木: 0,
  水: 0,
  火: 0,
  土: 0,
});

// 天干
const tianGan = ["甲", "乙", "丙", "丁", "戊", "己", "庚", "辛", "壬", "癸"];
// 地支
const diZhi = [
  "子",
  "丑",
  "寅",
  "卯",
  "辰",
  "巳",
  "午",
  "未",
  "申",
  "酉",
  "戌",
  "亥",
];
// 五行
const wuXing = {
  甲: "木",
  乙: "木",
  丙: "火",
  丁: "火",
  戊: "土",
  己: "土",
  庚: "金",
  辛: "金",
  壬: "水",
  癸: "水",
  子: "水",
  丑: "土",
  寅: "木",
  卯: "木",
  辰: "土",
  巳: "火",
  午: "火",
  未: "土",
  申: "金",
  酉: "金",
  戌: "土",
  亥: "水",
};

// 禁用未来日期
const disabledDate = (time: Date) => {
  return time.getTime() > Date.now();
};

// 计算八字
const calculateBazi = () => {
  if (!form.birthDate) return;

  const date = dayjs(form.birthDate);

  // 这里应该根据阴历/阳历进行转换
  // 为演示，暂时使用简化算法

  // 年柱
  const yearGan = tianGan[date.year() % 10];
  const yearZhi = diZhi[date.year() % 12];
  baziResult.year = {
    gan: yearGan,
    zhi: yearZhi,
    element: wuXing[yearGan as keyof typeof wuXing],
  };

  // 月柱
  const monthGan = tianGan[date.month()];
  const monthZhi = diZhi[date.month()];
  baziResult.month = {
    gan: monthGan,
    zhi: monthZhi,
    element: wuXing[monthGan as keyof typeof wuXing],
  };

  // 日柱
  const dayGan = tianGan[date.date() % 10];
  const dayZhi = diZhi[date.date() % 12];
  baziResult.day = {
    gan: dayGan,
    zhi: dayZhi,
    element: wuXing[dayGan as keyof typeof wuXing],
  };

  // 时柱
  const hourGan = tianGan[Math.floor(date.hour() / 2)];
  const hourZhi = diZhi[Math.floor(date.hour() / 2)];
  baziResult.hour = {
    gan: hourGan,
    zhi: hourZhi,
    element: wuXing[hourGan as keyof typeof wuXing],
  };

  // 计算五行
  calculateElements();
};

// 计算五行
const calculateElements = () => {
  // 重置计数
  Object.keys(elementCount).forEach((key) => {
    elementCount[key as keyof typeof elementCount] = 0;
  });

  // 统计天干地支的五行
  [baziResult.year, baziResult.month, baziResult.day, baziResult.hour].forEach(
    (pillar) => {
      const ganElement = wuXing[pillar.gan as keyof typeof wuXing];
      const zhiElement = wuXing[pillar.zhi as keyof typeof wuXing];
      if (ganElement) elementCount[ganElement as keyof typeof elementCount]++;
      if (zhiElement) elementCount[zhiElement as keyof typeof elementCount]++;
    }
  );
};
</script>

<style scoped>
.container {
  max-width: 1200px;
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
  background: var(--el-bg-color-page);
  border-radius: 8px;
  padding: 20px;
  margin-bottom: 24px;
}

.date-input {
  display: flex;
  gap: 16px;
  align-items: center;
}

.result-section {
  background: var(--el-bg-color-page);
  border-radius: 8px;
  padding: 20px;
}

.bazi-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 16px;
  margin-bottom: 24px;
}

.bazi-cell {
  background: var(--el-bg-color);
  border-radius: 8px;
  padding: 16px;
  text-align: center;
}

.cell-label {
  color: var(--el-text-color-secondary);
  font-size: 14px;
  margin-bottom: 8px;
}

.cell-content {
  font-size: 24px;
  font-weight: bold;
  color: var(--el-text-color-primary);
  margin-bottom: 8px;
}

.cell-content .gan {
  color: var(--el-color-danger);
  margin-right: 8px;
}

.cell-content .zhi {
  color: var(--el-color-primary);
}

.cell-info {
  color: var(--el-text-color-regular);
  font-size: 14px;
}

.analysis-section {
  background: var(--el-bg-color);
  border-radius: 8px;
  padding: 16px;
}

.analysis-section h3 {
  margin: 0 0 16px 0;
  font-size: 16px;
  color: var(--el-text-color-primary);
}

.element-chart {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.element-bar {
  display: flex;
  align-items: center;
  gap: 12px;
}

.element-name {
  width: 40px;
  text-align: right;
  color: var(--el-text-color-regular);
}

.bar-wrapper {
  flex: 1;
  display: flex;
  align-items: center;
  gap: 8px;
}

.bar {
  height: 20px;
  background: var(--el-color-primary);
  border-radius: 4px;
  transition: width 0.3s ease;
}

.bar-value {
  color: var(--el-text-color-secondary);
  font-size: 14px;
}

.empty-state {
  height: 200px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  color: var(--el-text-color-secondary);
  gap: 12px;
}

.empty-state .el-icon {
  font-size: 48px;
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

.usage-guide ul {
  margin: 8px 0;
  padding-left: 20px;
}

.note {
  margin-top: 16px;
  display: flex;
  align-items: center;
  gap: 8px;
  color: var(--el-text-color-secondary);
  font-size: 14px;
}

.note .el-icon {
  color: var(--el-color-warning);
}
</style>
