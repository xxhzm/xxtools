<template>
  <div class="container">
    <div class="nav-header">
      <el-button link @click="router.push('/')" class="back-btn">
        <el-icon><ArrowLeft /></el-icon>
        返回首页
      </el-button>
    </div>

    <div class="tool-header">
      <el-icon class="header-icon"><Odometer /></el-icon>
      <div class="header-text">
        <h1>单位转换</h1>
        <p class="subtitle">
          在线单位换算工具，支持长度、面积、体积、重量、温度等常用单位转换
        </p>
      </div>
    </div>

    <div class="tool-content">
      <div class="unit-type-select">
        <el-select v-model="currentType" placeholder="请选择单位类型">
          <el-option
            v-for="type in unitTypes"
            :key="type.value"
            :label="type.label"
            :value="type.value"
          />
        </el-select>
      </div>

      <div class="conversion-section">
        <div class="input-group">
          <el-input-number
            v-model="inputValue"
            :precision="precision"
            :step="1"
            :min="0"
            placeholder="输入数值"
            @change="convert"
          />
          <el-select
            v-model="fromUnit"
            placeholder="选择单位"
            @change="convert"
          >
            <el-option
              v-for="unit in currentUnits"
              :key="unit.value"
              :label="unit.label"
              :value="unit.value"
            />
          </el-select>
        </div>

        <div class="conversion-arrow">
          <el-icon><Right /></el-icon>
        </div>

        <div class="input-group">
          <el-input-number
            v-model="outputValue"
            :precision="precision"
            disabled
            placeholder="转换结果"
          />
          <el-select v-model="toUnit" placeholder="选择单位" @change="convert">
            <el-option
              v-for="unit in currentUnits"
              :key="unit.value"
              :label="unit.label"
              :value="unit.value"
            />
          </el-select>
        </div>
      </div>

      <div class="quick-convert">
        <el-button type="primary" plain @click="swapUnits">
          <el-icon><Sort /></el-icon>
          交换单位
        </el-button>
        <el-button @click="resetValues">
          <el-icon><RefreshRight /></el-icon>
          重置
        </el-button>
      </div>
    </div>

    <div class="usage-guide">
      <h2>使用说明</h2>
      <ol>
        <li>选择要转换的单位类型（长度、面积等）</li>
        <li>输入要转换的数值</li>
        <li>选择起始单位和目标单位</li>
        <li>自动显示转换结果</li>
        <li>可以点击交换按钮快速交换起始单位和目标单位</li>
      </ol>
      <div class="note">
        <el-icon><InfoFilled /></el-icon>
        <span>提示：数值保留4位小数，支持正数和负数</span>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, watch } from "vue";
import { useRouter } from "vue-router";
import {
  ArrowLeft,
  Right,
  Sort,
  RefreshRight,
  InfoFilled,
  Odometer,
} from "@element-plus/icons-vue";
import toolsData from "~/data/data.json";

const router = useRouter();

// 从data.json导入工具信息
const toolInfo = toolsData.tools.find((tool) => tool.id === 19);

// 更新页面标题和描述
const title = toolInfo?.title || "单位转换";
const description = toolInfo?.description || "在线单位换算工具";

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

// 单位类型定义
const unitTypes = [
  { label: "长度", value: "length" },
  { label: "面积", value: "area" },
  { label: "体积", value: "volume" },
  { label: "重量", value: "weight" },
  { label: "温度", value: "temperature" },
];

interface Unit {
  label: string;
  value: string;
  ratio?: number;
}

interface UnitType {
  [key: string]: Unit[];
}

// 单位定义
const units: UnitType = {
  length: [
    { label: "毫米(mm)", value: "mm", ratio: 1 },
    { label: "厘米(cm)", value: "cm", ratio: 10 },
    { label: "分米(dm)", value: "dm", ratio: 100 },
    { label: "米(m)", value: "m", ratio: 1000 },
    { label: "千米(km)", value: "km", ratio: 1000000 },
    { label: "英寸(in)", value: "in", ratio: 25.4 },
    { label: "英尺(ft)", value: "ft", ratio: 304.8 },
    { label: "码(yd)", value: "yd", ratio: 914.4 },
    { label: "英里(mi)", value: "mi", ratio: 1609344 },
  ],
  area: [
    { label: "平方毫米(mm²)", value: "mm2", ratio: 1 },
    { label: "平方厘米(cm²)", value: "cm2", ratio: 100 },
    { label: "平方分米(dm²)", value: "dm2", ratio: 10000 },
    { label: "平方米(m²)", value: "m2", ratio: 1000000 },
    { label: "公亩", value: "are", ratio: 100000000 },
    { label: "公顷(ha)", value: "ha", ratio: 10000000000 },
    { label: "平方千米(km²)", value: "km2", ratio: 1000000000000 },
  ],
  volume: [
    { label: "毫升(ml)", value: "ml", ratio: 1 },
    { label: "厘升(cl)", value: "cl", ratio: 10 },
    { label: "分升(dl)", value: "dl", ratio: 100 },
    { label: "升(l)", value: "l", ratio: 1000 },
    { label: "立方厘米(cm³)", value: "cm3", ratio: 1 },
    { label: "立方分米(dm³)", value: "dm3", ratio: 1000 },
    { label: "立方米(m³)", value: "m3", ratio: 1000000 },
  ],
  weight: [
    { label: "毫克(mg)", value: "mg", ratio: 1 },
    { label: "克(g)", value: "g", ratio: 1000 },
    { label: "千克(kg)", value: "kg", ratio: 1000000 },
    { label: "吨(t)", value: "t", ratio: 1000000000 },
    { label: "盎司(oz)", value: "oz", ratio: 28349.523125 },
    { label: "磅(lb)", value: "lb", ratio: 453592.37 },
  ],
  temperature: [
    { label: "摄氏度(°C)", value: "c" },
    { label: "华氏度(°F)", value: "f" },
    { label: "开尔文(K)", value: "k" },
  ],
};

const currentType = ref("length");
const inputValue = ref<number>();
const outputValue = ref<number>();
const fromUnit = ref("");
const toUnit = ref("");
const precision = ref(4);

// 当前选中类型的单位列表
const currentUnits = computed(() => {
  return units[currentType.value as keyof typeof units] || [];
});

// 温度转换特殊处理
const convertTemperature = (
  value: number,
  from: string,
  to: string
): number => {
  if (from === to) return value;

  // 先转换为摄氏度
  let celsius = value;
  if (from === "f") {
    celsius = ((value - 32) * 5) / 9;
  } else if (from === "k") {
    celsius = value - 273.15;
  }

  // 从摄氏度转换为目标单位
  if (to === "f") {
    return (celsius * 9) / 5 + 32;
  } else if (to === "k") {
    return celsius + 273.15;
  }
  return celsius;
};

// 单位转换
const convert = () => {
  if (!inputValue.value || !fromUnit.value || !toUnit.value) {
    outputValue.value = undefined;
    return;
  }

  if (currentType.value === "temperature") {
    outputValue.value = convertTemperature(
      inputValue.value,
      fromUnit.value,
      toUnit.value
    );
    return;
  }

  const unitList = units[currentType.value as keyof typeof units];
  const fromUnitData = unitList.find((u) => u.value === fromUnit.value);
  const toUnitData = unitList.find((u) => u.value === toUnit.value);

  if (!fromUnitData?.ratio || !toUnitData?.ratio) {
    outputValue.value = undefined;
    return;
  }

  outputValue.value =
    (inputValue.value * fromUnitData.ratio) / toUnitData.ratio;
};

// 交换单位
const swapUnits = () => {
  [fromUnit.value, toUnit.value] = [toUnit.value, fromUnit.value];
  convert();
};

// 重置
const resetValues = () => {
  inputValue.value = undefined;
  outputValue.value = undefined;
  fromUnit.value = "";
  toUnit.value = "";
};

// 监听类型变化
watch(currentType, () => {
  resetValues();
});
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

.unit-type-select {
  margin-bottom: 24px;
}

.conversion-section {
  display: flex;
  align-items: center;
  gap: 16px;
  margin-bottom: 24px;
}

.input-group {
  flex: 1;
  display: flex;
  gap: 8px;
}

.input-group :deep(.el-input-number) {
  flex: 1;
}

.input-group :deep(.el-select) {
  width: 140px;
}

.conversion-arrow {
  color: var(--el-text-color-secondary);
  font-size: 24px;
}

.quick-convert {
  display: flex;
  gap: 12px;
  justify-content: center;
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
  .container {
    padding: 15px;
  }

  .conversion-section {
    flex-direction: column;
  }

  .conversion-arrow {
    transform: rotate(90deg);
  }

  .input-group {
    width: 100%;
  }
}
</style>
