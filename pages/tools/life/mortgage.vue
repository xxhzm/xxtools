<template>
  <div class="container">
    <div class="nav-header">
      <el-button link @click="router.push('/')" class="back-btn">
        <el-icon><ArrowLeft /></el-icon>
        返回首页
      </el-button>
    </div>

    <div class="tool-header">
      <el-icon class="header-icon"><House /></el-icon>
      <div class="header-text">
        <h1>{{ title }}</h1>
        <p class="subtitle">{{ description }}</p>
      </div>
    </div>

    <div class="tool-content">
      <div class="calculator-section">
        <div class="input-section">
          <el-form :model="form" label-width="120px">
            <el-form-item label="贷款金额">
              <el-input-number
                v-model="form.amount"
                :min="1"
                :max="10000"
                :step="1"
                :precision="0"
                @change="calculate"
              >
                <template #append>万元</template>
              </el-input-number>
            </el-form-item>
            <el-form-item label="贷款期限">
              <el-input-number
                v-model="form.years"
                :min="1"
                :max="30"
                :step="1"
                @change="calculate"
              >
                <template #append>年</template>
              </el-input-number>
            </el-form-item>
            <el-form-item label="年利率">
              <el-input-number
                v-model="form.rate"
                :min="0.1"
                :max="20"
                :step="0.1"
                :precision="2"
                @change="calculate"
              >
                <template #append>%</template>
              </el-input-number>
            </el-form-item>
            <el-form-item label="还款方式">
              <el-radio-group v-model="form.type" @change="calculate">
                <el-radio label="equal">等额本息</el-radio>
                <el-radio label="principal">等额本金</el-radio>
              </el-radio-group>
            </el-form-item>
          </el-form>
        </div>

        <div class="result-section">
          <div class="result-header">
            <h3>计算结果</h3>
          </div>
          <div class="result-grid">
            <div class="result-item">
              <div class="result-label">贷款总额</div>
              <div class="result-value">
                {{ formatMoney(form.amount * 10000) }}元
                <div class="chinese-money">
                  {{ formatChineseMoney(form.amount * 10000) }}元
                </div>
              </div>
              <div class="result-desc">您需要借入的资金总额</div>
            </div>
            <div class="result-item">
              <div class="result-label">
                {{ form.type === "equal" ? "月供" : "首月还款" }}
              </div>
              <div class="result-value">
                {{ formatMoney(result.monthlyPayment) }}元
                <div class="chinese-money">
                  {{ formatChineseMoney(result.monthlyPayment) }}元
                </div>
              </div>
              <div class="result-desc">
                {{
                  form.type === "equal"
                    ? "每月需要还款的固定金额"
                    : "第一个月需要还款的金额，后续逐月递减"
                }}
              </div>
            </div>
            <div class="result-item">
              <div class="result-label">总利息</div>
              <div class="result-value">
                {{ formatMoney(result.totalInterest) }}元
                <div class="chinese-money">
                  {{ formatChineseMoney(result.totalInterest) }}元
                </div>
              </div>
              <div class="result-desc">整个还款期限内支付的利息总和</div>
            </div>
            <div class="result-item">
              <div class="result-label">还款总额</div>
              <div class="result-value">
                {{ formatMoney(result.totalPayment) }}元
                <div class="chinese-money">
                  {{ formatChineseMoney(result.totalPayment) }}元
                </div>
              </div>
              <div class="result-desc">
                本金和利息的总和，即您最终需要还款的总金额
              </div>
            </div>
          </div>

          <div class="monthly-detail" v-if="form.type === 'principal'">
            <div class="detail-header">
              <h4>月供递减金额</h4>
              <div class="detail-value">
                {{ formatMoney(result.monthlyDecrease) }}元
              </div>
            </div>
            <div class="detail-text">
              每月递减说明：等额本金方式下，每月还款金额会逐月减少，月供递减金额为每月还款金额的减少值。
            </div>
          </div>

          <el-collapse class="payment-schedule">
            <el-collapse-item title="查看还款计划" name="1">
              <el-table
                :data="result.schedule"
                height="300"
                style="width: 100%"
              >
                <el-table-column prop="month" label="期数" width="80" />
                <el-table-column prop="payment" label="还款金额">
                  <template #default="scope">
                    {{ formatMoney(scope.row.payment) }}
                  </template>
                </el-table-column>
                <el-table-column prop="principal" label="本金">
                  <template #default="scope">
                    {{ formatMoney(scope.row.principal) }}
                  </template>
                </el-table-column>
                <el-table-column prop="interest" label="利息">
                  <template #default="scope">
                    {{ formatMoney(scope.row.interest) }}
                  </template>
                </el-table-column>
                <el-table-column prop="remaining" label="剩余本金">
                  <template #default="scope">
                    {{ formatMoney(scope.row.remaining) }}
                  </template>
                </el-table-column>
              </el-table>
            </el-collapse-item>
          </el-collapse>
        </div>
      </div>
    </div>

    <div class="usage-guide">
      <h2>使用说明</h2>
      <ol>
        <li>
          输入贷款信息：
          <ul>
            <li>贷款金额：输入需要贷款的金额（单位：万元）</li>
            <li>贷款期限：选择贷款年限（1-30年）</li>
            <li>年利率：输入贷款年利率（如：4.85%）</li>
          </ul>
        </li>
        <li>
          选择还款方式：
          <ul>
            <li>等额本息：每月还款金额相同，其中本金逐月递增，利息逐月递减</li>
            <li>等额本金：每月本金还款额固定，总还款额逐月递减</li>
          </ul>
        </li>
        <li>查看计算结果：包括月供、总利息、还款总额等信息</li>
        <li>展开还款计划可查看每月详细的还款信息</li>
      </ol>
      <div class="note">
        <el-icon><InfoFilled /></el-icon>
        <span>提示：本计算结果仅供参考，实际贷款情况请以银行审批为准</span>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive } from "vue";
import { useRouter } from "vue-router";
import { ArrowLeft, House, InfoFilled } from "@element-plus/icons-vue";
import toolsData from "~/data/data.json";

const router = useRouter();
const toolInfo = toolsData.tools.find((tool) => tool.id === 37);

// 更新页面标题和描述
const title = toolInfo?.title || "房贷计算";
const description = toolInfo?.description || "房贷计算器";

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
  amount: 100, // 贷款金额（万元）
  years: 30, // 贷款年限
  rate: 4.85, // 年利率
  type: "equal", // 还款方式：equal-等额本息，principal-等额本金
});

// 计算结果
interface PaymentSchedule {
  month: number;
  payment: number;
  principal: number;
  interest: number;
  remaining: number;
}

interface CalculationResult {
  monthlyPayment: number;
  totalInterest: number;
  totalPayment: number;
  monthlyDecrease: number;
  schedule: PaymentSchedule[];
}

const result = reactive<CalculationResult>({
  monthlyPayment: 0,
  totalInterest: 0,
  totalPayment: 0,
  monthlyDecrease: 0,
  schedule: [],
});

// 计算还款信息
const calculate = () => {
  const principal = form.amount * 10000; // 贷款金额（元）
  const months = form.years * 12; // 还款月数
  const monthlyRate = form.rate / 100 / 12; // 月利率

  if (form.type === "equal") {
    // 等额本息
    // 月供 = 本金 × 月利率 × (1 + 月利率)^还款月数 ÷ [(1 + 月利率)^还款月数 - 1]
    const monthlyPayment =
      (principal * monthlyRate * Math.pow(1 + monthlyRate, months)) /
      (Math.pow(1 + monthlyRate, months) - 1);

    result.monthlyPayment = monthlyPayment;
    result.totalPayment = monthlyPayment * months;
    result.totalInterest = result.totalPayment - principal;
    result.monthlyDecrease = 0;

    // 生成还款计划
    result.schedule = [];
    let remainingPrincipal = principal;
    for (let i = 1; i <= months; i++) {
      const interest = remainingPrincipal * monthlyRate;
      const principalPayment = monthlyPayment - interest;
      remainingPrincipal -= principalPayment;

      result.schedule.push({
        month: i,
        payment: monthlyPayment,
        principal: principalPayment,
        interest: interest,
        remaining: Math.max(0, remainingPrincipal),
      });
    }
  } else {
    // 等额本金
    const monthlyPrincipal = principal / months; // 每月应还本金
    const firstMonthPayment = monthlyPrincipal + principal * monthlyRate; // 首月还款额

    result.monthlyPayment = firstMonthPayment;
    result.monthlyDecrease = monthlyPrincipal * monthlyRate; // 每月递减金额

    // 生成还款计划
    result.schedule = [];
    let totalPayment = 0;
    let totalInterest = 0;
    let remainingPrincipal = principal;

    for (let i = 1; i <= months; i++) {
      const interest = remainingPrincipal * monthlyRate;
      const payment = monthlyPrincipal + interest;
      remainingPrincipal -= monthlyPrincipal;

      totalPayment += payment;
      totalInterest += interest;

      result.schedule.push({
        month: i,
        payment: payment,
        principal: monthlyPrincipal,
        interest: interest,
        remaining: Math.max(0, remainingPrincipal),
      });
    }

    result.totalPayment = totalPayment;
    result.totalInterest = totalInterest;
  }
};

// 格式化金额
const formatMoney = (value: number) => {
  return value.toFixed(2).replace(/\B(?=(\d{3})+(?!\d))/g, ",");
};

// 格式化中文金额显示
const formatChineseMoney = (value: number) => {
  if (value >= 100000000) {
    return `${(value / 100000000).toFixed(2)}亿`;
  } else if (value >= 10000) {
    return `${(value / 10000).toFixed(2)}万`;
  }
  return `${value.toFixed(2)}`;
};

// 初始计算
calculate();
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

.calculator-section {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 24px;
}

.input-section {
  padding: 20px;
  background: var(--el-bg-color-page);
  border-radius: 8px;
}

.result-section {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.result-header {
  margin-bottom: 16px;
}

.result-header h3 {
  font-size: 18px;
  margin: 0;
  color: var(--el-text-color-primary);
}

.result-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 16px;
}

.result-item {
  background: var(--el-bg-color-page);
  padding: 16px;
  border-radius: 8px;
  text-align: center;
}

.result-label {
  color: var(--el-text-color-secondary);
  font-size: 14px;
  margin-bottom: 8px;
}

.result-value {
  color: var(--el-color-primary);
  font-size: 20px;
  font-weight: 500;
  margin-bottom: 4px;
}

.chinese-money {
  color: var(--el-text-color-regular);
  font-size: 14px;
  margin-top: 2px;
}

.result-desc {
  color: var(--el-text-color-secondary);
  font-size: 12px;
  line-height: 1.4;
}

.monthly-detail {
  background: var(--el-bg-color-page);
  padding: 16px;
  border-radius: 8px;
  margin-top: 16px;
}

.detail-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 12px;
}

.detail-header h4 {
  margin: 0;
  font-size: 14px;
  color: var(--el-text-color-primary);
}

.detail-value {
  color: var(--el-color-primary);
  font-weight: 500;
}

.detail-text {
  font-size: 12px;
  color: var(--el-text-color-secondary);
  line-height: 1.5;
}

.payment-schedule {
  margin-top: 16px;
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

  .calculator-section {
    grid-template-columns: 1fr;
  }

  .result-grid {
    grid-template-columns: 1fr;
  }
}
</style>
