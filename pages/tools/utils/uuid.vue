<template>
  <div class="container">
    <div class="nav-header">
      <el-button link @click="router.push('/')" class="back-btn">
        <el-icon><ArrowLeft /></el-icon>
        返回首页
      </el-button>
    </div>

    <div class="tool-header">
      <el-icon class="header-icon"><Operation /></el-icon>
      <div class="header-text">
        <h1>UUID生成</h1>
        <p class="subtitle">
          生成随机的UUID/GUID标识符，支持批量生成和自定义格式
        </p>
      </div>
    </div>

    <div class="tool-content">
      <div class="options-section">
        <div class="option-row">
          <span class="option-label">生成数量：</span>
          <el-input-number
            v-model="count"
            :min="1"
            :max="100"
            size="default"
            @change="generateUUIDs"
          />
        </div>

        <div class="option-row">
          <span class="option-label">UUID版本：</span>
          <el-radio-group v-model="version" @change="generateUUIDs">
            <el-radio-button label="4">v4 (随机)</el-radio-button>
            <el-radio-button label="1">v1 (时间)</el-radio-button>
          </el-radio-group>
        </div>

        <div class="option-row">
          <span class="option-label">大小写：</span>
          <el-radio-group v-model="caseType" @change="generateUUIDs">
            <el-radio-button label="lower">小写</el-radio-button>
            <el-radio-button label="upper">大写</el-radio-button>
          </el-radio-group>
        </div>

        <div class="option-row">
          <span class="option-label">分隔符：</span>
          <el-radio-group v-model="separator" @change="generateUUIDs">
            <el-radio-button label="-">连字符</el-radio-button>
            <el-radio-button label="">无分隔符</el-radio-button>
          </el-radio-group>
        </div>

        <el-button type="primary" class="generate-btn" @click="generateUUIDs">
          重新生成
        </el-button>
      </div>

      <div v-if="uuids.length > 0" class="result-section">
        <div class="uuid-list">
          <div v-for="(uuid, index) in uuids" :key="index" class="uuid-item">
            <el-input v-model="uuids[index]" readonly size="default">
              <template #append>
                <el-button @click="copyUUID(uuid)">
                  <el-icon><CopyDocument /></el-icon>
                  复制
                </el-button>
              </template>
            </el-input>
          </div>
        </div>
        <div class="batch-actions">
          <el-button type="primary" @click="copyAll">
            <el-icon><CopyDocument /></el-icon>
            复制全部
          </el-button>
        </div>
      </div>
    </div>

    <div class="usage-guide">
      <h2>使用说明</h2>
      <ol>
        <li>选择需要生成的UUID数量（1-100个）</li>
        <li>选择UUID版本（v4随机生成或v1基于时间生成）</li>
        <li>选择输出格式（大小写、是否包含分隔符）</li>
        <li>点击"重新生成"按钮生成新的UUID</li>
        <li>点击单个UUID后的"复制"按钮复制，或使用"复制全部"</li>
      </ol>
      <div class="note">
        <el-icon><InfoFilled /></el-icon>
        <span
          >提示：UUID是通用唯一识别码，可用于数据库主键、临时文件名等场景</span
        >
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from "vue";
import { ElMessage } from "element-plus";
import { useRouter } from "vue-router";
import {
  Operation,
  CopyDocument,
  InfoFilled,
  ArrowLeft,
} from "@element-plus/icons-vue";
import toolsData from "~/data/data.json";

const router = useRouter();
const count = ref(1);
const version = ref("4");
const caseType = ref("lower");
const separator = ref("-");
const uuids = ref<string[]>([]);

// 从data.json导入工具信息
const toolInfo = toolsData.tools.find((tool) => tool.id === 13);

// 更新页面标题和描述
const title = toolInfo?.title || "UUID生成";
const description =
  toolInfo?.description ||
  "生成随机的UUID/GUID标识符，支持批量生成和自定义格式";

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

// 生成随机十六进制数
const randomHex = (length: number) => {
  const bytes = new Uint8Array(length);
  crypto.getRandomValues(bytes);
  return Array.from(bytes)
    .map((b) => b.toString(16).padStart(2, "0"))
    .join("")
    .slice(0, length);
};

// 生成时间戳十六进制数
const getTimestampHex = () => {
  const time = Date.now();
  return time.toString(16).padStart(12, "0");
};

// 生成v4版本UUID（完全随机）
const generateV4 = () => {
  const uuid = randomHex(32);
  // 设置版本号为4
  const arr = uuid.split("");
  arr[12] = "4";
  // 设置变体位
  arr[16] = ((parseInt(arr[16], 16) & 0x3) | 0x8).toString(16);

  return [
    arr.slice(0, 8).join(""),
    arr.slice(8, 12).join(""),
    arr.slice(12, 16).join(""),
    arr.slice(16, 20).join(""),
    arr.slice(20, 32).join(""),
  ].join("-");
};

// 生成v1版本UUID（基于时间）
const generateV1 = () => {
  const timestamp = getTimestampHex();
  const clockSeq = randomHex(4);
  const node = randomHex(12);

  // 设置版本号为1
  const timeHigh = timestamp.slice(0, 3);
  const timeMid = timestamp.slice(3, 7);
  const timeHighAndVersion = (parseInt(timeHigh + "0", 16) | 0x1000)
    .toString(16)
    .slice(1);

  // 设置变体位
  const clockSeqHighAndReserved = (parseInt(clockSeq.slice(0, 2), 16) | 0x80)
    .toString(16)
    .padStart(2, "0");
  const clockSeqLow = clockSeq.slice(2, 4);

  return [
    timeMid + timestamp.slice(7),
    timeHighAndVersion,
    "1" + timeHigh,
    clockSeqHighAndReserved + clockSeqLow,
    node,
  ].join("-");
};

const generateUUIDs = () => {
  uuids.value = [];
  for (let i = 0; i < count.value; i++) {
    let uuid = version.value === "4" ? generateV4() : generateV1();

    // 处理分隔符
    if (separator.value === "") {
      uuid = uuid.replace(/-/g, "");
    }

    // 处理大小写
    uuid = caseType.value === "upper" ? uuid.toUpperCase() : uuid.toLowerCase();

    uuids.value.push(uuid);
  }
};

const copyUUID = async (uuid: string) => {
  try {
    await navigator.clipboard.writeText(uuid);
    ElMessage.success("UUID已复制到剪贴板");
  } catch (err) {
    try {
      const textarea = document.createElement("textarea");
      textarea.value = uuid;
      textarea.style.position = "fixed";
      textarea.style.opacity = "0";
      document.body.appendChild(textarea);
      textarea.select();

      const successful = document.execCommand("copy");
      document.body.removeChild(textarea);

      if (successful) {
        ElMessage.success("UUID已复制到剪贴板");
      } else {
        throw new Error("复制失败");
      }
    } catch (fallbackErr) {
      ElMessage({
        message: "复制失败，请手动选择并使用 Ctrl+C（或 Command+C）复制",
        type: "error",
        duration: 5000,
      });
    }
  }
};

const copyAll = async () => {
  const text = uuids.value.join("\n");
  try {
    await navigator.clipboard.writeText(text);
    ElMessage.success("所有UUID已复制到剪贴板");
  } catch (err) {
    try {
      const textarea = document.createElement("textarea");
      textarea.value = text;
      textarea.style.position = "fixed";
      textarea.style.opacity = "0";
      document.body.appendChild(textarea);
      textarea.select();

      const successful = document.execCommand("copy");
      document.body.removeChild(textarea);

      if (successful) {
        ElMessage.success("所有UUID已复制到剪贴板");
      } else {
        throw new Error("复制失败");
      }
    } catch (fallbackErr) {
      ElMessage({
        message: "复制失败，请手动选择并使用 Ctrl+C（或 Command+C）复制",
        type: "error",
        duration: 5000,
      });
    }
  }
};

// 初始生成一个UUID
generateUUIDs();
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

.options-section {
  margin-bottom: 24px;
}

.option-row {
  display: flex;
  align-items: center;
  gap: 12px;
  margin-bottom: 16px;
}

.option-label {
  color: var(--el-text-color-regular);
  min-width: 80px;
}

.generate-btn {
  width: 100%;
  margin-top: 8px;
}

.result-section {
  padding-top: 24px;
  border-top: 1px solid var(--el-border-color-light);
}

.uuid-list {
  display: flex;
  flex-direction: column;
  gap: 12px;
  margin-bottom: 20px;
}

.uuid-item :deep(.el-input__wrapper) {
  font-family: monospace;
}

.batch-actions {
  display: flex;
  justify-content: center;
  gap: 16px;
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

  .option-row {
    flex-direction: column;
    align-items: flex-start;
    gap: 8px;
  }

  .option-label {
    margin-bottom: 4px;
  }

  .el-radio-group {
    width: 100%;
  }

  :deep(.el-radio-button__inner) {
    width: 100%;
  }
}
</style>
