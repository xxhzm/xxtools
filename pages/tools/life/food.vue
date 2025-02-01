<template>
  <div class="container">
    <div class="nav-header">
      <el-button link @click="router.push('/')" class="back-btn">
        <el-icon><ArrowLeft /></el-icon>
        返回首页
      </el-button>
    </div>

    <div class="tool-header">
      <el-icon class="header-icon"><Food /></el-icon>
      <div class="header-text">
        <h1>{{ title }}</h1>
        <p class="subtitle">{{ description }}</p>
      </div>
    </div>

    <div class="tool-content">
      <div class="food-result" v-if="currentFood">
        <div class="result-animation" :class="{ active: isAnimating }">
          <h2
            :class="{
              'welcome-text':
                !isAnimating && currentFood === '点击下方按钮开始随机',
            }"
          >
            {{ currentFood }}
          </h2>
        </div>
        <el-button
          type="primary"
          size="large"
          @click="startRandom"
          :loading="isAnimating"
        >
          <el-icon><RefreshRight /></el-icon>
          {{ currentFood === "点击下方按钮开始随机" ? "开始随机" : "换一个" }}
        </el-button>
      </div>

      <div class="food-lists">
        <el-tabs v-model="activeTab">
          <el-tab-pane label="系统菜单" name="system">
            <div class="list-header">
              <h3>系统内置菜单</h3>
              <div class="actions">
                <el-button size="small" @click="useSystemList">
                  <el-icon><Select /></el-icon>
                  使用此列表
                </el-button>
                <el-button size="small" @click="addAllToCustom">
                  <el-icon><Plus /></el-icon>
                  全部添加到自定义
                </el-button>
              </div>
            </div>
            <div class="food-tags">
              <el-tag
                v-for="food in defaultSystemFoodList"
                :key="food"
                class="food-tag"
              >
                {{ food }}
                <el-button
                  class="tag-action-btn"
                  size="small"
                  @click.stop="addToCustomList(food)"
                >
                  <el-icon><Plus /></el-icon>
                </el-button>
              </el-tag>
            </div>
          </el-tab-pane>

          <el-tab-pane label="我的菜单" name="custom">
            <div class="list-header">
              <h3>自定义菜单</h3>
              <div class="actions">
                <el-button size="small" @click="useCustomList">
                  <el-icon><Select /></el-icon>
                  使用此列表
                </el-button>
                <el-button size="small" @click="clearCustomList">
                  <el-icon><Delete /></el-icon>
                  清空列表
                </el-button>
              </div>
            </div>
            <div class="custom-input">
              <el-input
                v-model="newFood"
                placeholder="输入菜品名称"
                @keyup.enter="addCustomFood"
              >
                <template #append>
                  <el-button @click="addCustomFood">
                    <el-icon><Plus /></el-icon>
                    添加
                  </el-button>
                </template>
              </el-input>
            </div>
            <div class="food-tags">
              <el-tag
                v-for="food in customFoodList"
                :key="food"
                class="food-tag"
              >
                {{ food }}
                <el-button
                  class="tag-action-btn"
                  size="small"
                  @click.stop="removeCustomFood(food)"
                >
                  <el-icon><Close /></el-icon>
                </el-button>
              </el-tag>
            </div>
          </el-tab-pane>
        </el-tabs>
      </div>
    </div>

    <div class="usage-guide">
      <h2>使用说明</h2>
      <ol>
        <li>点击"换一个"按钮随机推荐一个菜品</li>
        <li>可以选择使用系统内置菜单或自定义菜单</li>
        <li>在系统菜单中，点击菜品旁的加号可以将其添加到自定义菜单</li>
        <li>
          也可以点击"全部添加到自定义"按钮，将所有系统菜品添加到自定义菜单
        </li>
        <li>在"我的菜单"中可以添加新菜品或删除不需要的菜品</li>
      </ol>
      <div class="note">
        <el-icon><InfoFilled /></el-icon>
        <span
          >提示：建议将常吃的系统菜品添加到自定义菜单，这样更容易管理和随机</span
        >
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from "vue";
import { useRouter } from "vue-router";
import {
  ArrowLeft,
  Food,
  RefreshRight,
  Select,
  Delete,
  Plus,
  InfoFilled,
  Close,
} from "@element-plus/icons-vue";
import { ElMessage } from "element-plus";
import toolsData from "~/data/data.json";

const router = useRouter();
const toolInfo = toolsData.tools.find((tool) => tool.id === 30);

// 更新页面标题和描述
const title = toolInfo?.title || "今天吃什么";
const description = toolInfo?.description || "随机推荐今天吃什么";

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

// 系统内置菜单
const defaultSystemFoodList = [
  // 中餐
  "红烧肉",
  "糖醋排骨",
  "鱼香肉丝",
  "宫保鸡丁",
  "麻婆豆腐",
  "水煮肉片",
  "回锅肉",
  "青椒肉丝",
  "西红柿炒蛋",
  "炸酱面",
  "重庆小面",
  "兰州拉面",
  "酸菜鱼",
  "东坡肉",
  "小龙虾",
  "毛血旺",
  "辣子鸡",
  "鱼香茄子",
  "干锅牛肉",
  "葱爆羊肉",
  "蒜蓉虾",
  "酱牛肉",
  "红烧排骨",
  "啤酒鸭",
  "清蒸鱼",
  // 快餐
  "肯德基",
  "麦当劳",
  "必胜客",
  "华莱士",
  "德克士",
  "汉堡王",
  "真功夫",
  "永和大王",
  "味千拉面",
  "吉野家",
  // 地方小吃
  "沙县小吃",
  "生煎包",
  "饺子",
  "肉夹馍",
  "煎饼果子",
  "炒面",
  "炒米粉",
  "烧烤",
  "麻辣烫",
  "米线",
  "过桥米线",
  "酸辣粉",
  "担担面",
  "羊肉泡馍",
  "牛肉面",
  // 其他
  "火锅",
  "烤肉",
  "寿司",
  "盖浇饭",
  "黄焖鸡米饭",
  "韩式炸鸡",
  "冒菜",
  "串串香",
  "麻辣香锅",
  "烤鱼",
  "石锅拌饭",
  "便当",
  "自助餐",
  "海底捞",
  "大排档",
];

// 移除不需要的状态和方法
const systemFoodList = ref<string[]>([...defaultSystemFoodList]);

// 自定义菜单
const customFoodList = ref<string[]>([]);
const activeTab = ref("system");
const currentFood = ref("");
const isAnimating = ref(false);
const newFood = ref("");

// 从localStorage加载菜单
onMounted(() => {
  const savedCustomList = localStorage.getItem("customFoodList");
  if (savedCustomList) {
    customFoodList.value = JSON.parse(savedCustomList);
  }
  currentFood.value = "点击下方按钮开始随机";
});

// 保存自定义菜单到localStorage
const saveCustomList = () => {
  localStorage.setItem("customFoodList", JSON.stringify(customFoodList.value));
};

// 添加自定义菜品
const addCustomFood = () => {
  const food = newFood.value.trim();
  if (!food) {
    ElMessage.warning("请输入菜品名称");
    return;
  }
  if (customFoodList.value.includes(food)) {
    ElMessage.warning("该菜品已存在");
    return;
  }
  customFoodList.value.push(food);
  saveCustomList();
  newFood.value = "";
  ElMessage.success("添加成功");
};

// 移除自定义菜品
const removeCustomFood = (food: string) => {
  const index = customFoodList.value.indexOf(food);
  if (index > -1) {
    customFoodList.value.splice(index, 1);
    saveCustomList();
  }
};

// 清空自定义列表
const clearCustomList = () => {
  customFoodList.value = [];
  saveCustomList();
  ElMessage.success("清空成功");
};

// 使用系统列表
const useSystemList = () => {
  activeTab.value = "system";
  ElMessage.success("已切换到系统菜单");
};

// 使用自定义列表
const useCustomList = () => {
  if (customFoodList.value.length === 0) {
    ElMessage.warning("自定义菜单为空，请先添加菜品");
    return;
  }
  activeTab.value = "custom";
  ElMessage.success("已切换到自定义菜单");
};

// 添加到自定义菜单
const addToCustomList = (food: string) => {
  if (customFoodList.value.includes(food)) {
    ElMessage.warning("该菜品已在自定义菜单中");
    return;
  }
  customFoodList.value.push(food);
  saveCustomList();
  ElMessage.success(`已将"${food}"添加到自定义菜单`);
  // 自动切换到自定义菜单
  activeTab.value = "custom";
};

// 添加所有系统菜单到自定义菜单
const addAllToCustom = () => {
  const newItems = defaultSystemFoodList.filter(
    (food) => !customFoodList.value.includes(food)
  );
  if (newItems.length === 0) {
    ElMessage.warning("所有系统菜品已在自定义菜单中");
    return;
  }
  customFoodList.value.push(...newItems);
  saveCustomList();
  ElMessage.success(`已添加${newItems.length}个菜品到自定义菜单`);
  // 自动切换到自定义菜单
  activeTab.value = "custom";
};

// 随机选择
const startRandom = () => {
  if (isAnimating.value) return;

  const list =
    activeTab.value === "system" ? defaultSystemFoodList : customFoodList.value;
  if (list.length === 0) {
    ElMessage.warning("当前菜单为空，请先添加菜品");
    return;
  }

  isAnimating.value = true;
  let count = 0;
  const maxCount = 20; // 动画次数
  const interval = setInterval(() => {
    const randomIndex = Math.floor(Math.random() * list.length);
    currentFood.value = list[randomIndex];
    count++;

    if (count >= maxCount) {
      clearInterval(interval);
      isAnimating.value = false;
    }
  }, 100);
};

// 初始化时随机一个
onMounted(() => {
  startRandom();
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

.food-result {
  text-align: center;
  margin-bottom: 32px;
  padding: 32px;
  background: var(--el-color-primary-light-9);
  border-radius: 8px;
}

.result-animation {
  margin-bottom: 24px;
  transition: transform 0.1s ease-in-out;
}

.result-animation.active {
  transform: scale(1.1);
}

.result-animation h2 {
  font-size: 32px;
  color: var(--el-color-primary);
  margin: 0;
}

.welcome-text {
  font-size: 24px;
  color: var(--el-text-color-secondary);
}

.food-lists {
  margin-top: 24px;
}

.list-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 16px;
}

.list-header h3 {
  font-size: 16px;
  color: var(--el-text-color-primary);
  margin: 0;
}

.actions {
  display: flex;
  gap: 8px;
}

.custom-input {
  margin-bottom: 16px;
}

.food-tags {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
  padding: 16px;
  background: var(--el-fill-color-light);
  border-radius: 4px;
  min-height: 100px;
}

.food-tag {
  font-size: 14px;
  position: relative;
  padding-right: 28px;
  margin: 4px;
  height: 28px;
  line-height: 26px;
}

.tag-action-btn {
  position: absolute;
  right: 0;
  top: 50%;
  transform: translateY(-50%);
  height: 28px;
  width: 28px;
  padding: 0;
  border: none;
  background: transparent;
  color: var(--el-text-color-secondary);
  transition: all 0.3s;
  display: flex;
  align-items: center;
  justify-content: center;
}

.tag-action-btn:hover {
  color: var(--el-color-primary);
  background: transparent;
}

.tag-action-btn .el-icon {
  font-size: 12px;
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

  .food-result {
    padding: 20px;
  }

  .result-animation h2 {
    font-size: 28px;
  }

  .actions {
    flex-direction: column;
    gap: 8px;
  }

  .list-header {
    flex-direction: column;
    align-items: flex-start;
    gap: 8px;
  }
}
</style>
