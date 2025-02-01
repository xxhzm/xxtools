<template>
  <div class="container">
    <div class="nav-header">
      <el-button link @click="router.push('/')" class="back-btn">
        <el-icon><ArrowLeft /></el-icon>
        返回首页
      </el-button>
    </div>

    <div class="tool-header">
      <el-icon class="header-icon"><Edit /></el-icon>
      <div class="header-text">
        <h1>CSS工具</h1>
        <p class="subtitle">
          在线CSS代码处理工具，支持美化、净化、整理、优化和压缩
        </p>
      </div>
    </div>

    <div class="tool-content">
      <div class="editor-section">
        <div class="editor-header">
          <h3>输入CSS代码</h3>
          <div class="actions">
            <el-button size="small" @click="clearInput">
              <el-icon><Delete /></el-icon>
              清空
            </el-button>
            <el-button size="small" @click="loadExample">
              <el-icon><DocumentAdd /></el-icon>
              加载示例
            </el-button>
          </div>
        </div>
        <el-input
          v-model="cssInput"
          type="textarea"
          :rows="10"
          placeholder="请输入CSS代码..."
          @input="handleInput"
        />
      </div>

      <div class="options-section">
        <div class="operation-type">
          <el-radio-group v-model="operationType" @change="processCSS">
            <el-radio-button label="beautify">美化</el-radio-button>
            <el-radio-button label="purify">净化</el-radio-button>
            <el-radio-button label="organize">整理</el-radio-button>
            <el-radio-button label="optimize">优化</el-radio-button>
            <el-radio-button label="minify">压缩</el-radio-button>
          </el-radio-group>
        </div>

        <div class="operation-options" v-if="operationType === 'beautify'">
          <el-form :model="beautifyOptions" label-width="120px" size="small">
            <el-form-item label="缩进">
              <el-radio-group
                v-model="beautifyOptions.indent"
                @change="processCSS"
              >
                <el-radio label="2">2空格</el-radio>
                <el-radio label="4">4空格</el-radio>
                <el-radio label="tab">Tab</el-radio>
              </el-radio-group>
            </el-form-item>
            <el-form-item label="每行最大字符">
              <el-input-number
                v-model="beautifyOptions.maxLine"
                :min="60"
                :max="200"
                @change="processCSS"
              />
            </el-form-item>
          </el-form>
        </div>

        <div class="operation-options" v-if="operationType === 'organize'">
          <el-form :model="organizeOptions" label-width="120px" size="small">
            <el-form-item label="排序方式">
              <el-select
                v-model="organizeOptions.sortOrder"
                @change="processCSS"
              >
                <el-option label="按字母顺序" value="alphabetical" />
                <el-option label="按属性类型" value="grouped" />
              </el-select>
            </el-form-item>
          </el-form>
        </div>
      </div>

      <div class="result-section">
        <div class="result-header">
          <h3>处理结果</h3>
          <div class="actions">
            <el-button size="small" type="primary" @click="copyResult">
              <el-icon><DocumentCopy /></el-icon>
              复制结果
            </el-button>
          </div>
        </div>
        <el-input
          v-model="cssOutput"
          type="textarea"
          :rows="10"
          readonly
          placeholder="处理后的CSS代码将显示在这里..."
        />
      </div>
    </div>

    <div class="usage-guide">
      <h2>功能说明</h2>
      <ul>
        <li>
          <strong>美化：</strong>格式化代码，使之容易阅读，支持自定义缩进和行宽
        </li>
        <li><strong>净化：</strong>将代码单行化，并去除注释</li>
        <li>
          <strong>整理：</strong
          >按照一定的顺序（字母顺序或属性类型），重新排列CSS属性
        </li>
        <li><strong>优化：</strong>将CSS的长属性值优化为简写的形式</li>
        <li>
          <strong>压缩：</strong>将代码最小化，去除空格和换行，加快加载速度
        </li>
      </ul>
      <div class="note">
        <el-icon><InfoFilled /></el-icon>
        <span>提示：所有处理都在本地进行，不会上传到服务器</span>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive } from "vue";
import { useRouter } from "vue-router";
import {
  ArrowLeft,
  Edit,
  Delete,
  DocumentAdd,
  DocumentCopy,
  InfoFilled,
} from "@element-plus/icons-vue";
import { ElMessage } from "element-plus";
import toolsData from "~/data/data.json";

const router = useRouter();

// 从data.json导入工具信息
const toolInfo = toolsData.tools.find((tool) => tool.id === 24);

// 更新页面标题和描述
const title = toolInfo?.title || "CSS工具";
const description = toolInfo?.description || "在线CSS代码处理工具";

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

const cssInput = ref("");
const cssOutput = ref("");
const operationType = ref("beautify");

// 美化选项
const beautifyOptions = reactive({
  indent: "2",
  maxLine: 80,
});

// 整理选项
const organizeOptions = reactive({
  sortOrder: "grouped",
});

// CSS属性分组（用于按类型排序）
const cssPropertyGroups = {
  positioning: ["position", "top", "right", "bottom", "left", "z-index"],
  display: ["display", "flex", "grid", "gap", "visibility", "opacity"],
  boxModel: [
    "margin",
    "padding",
    "width",
    "height",
    "box-sizing",
    "border",
    "border-radius",
  ],
  typography: [
    "font",
    "font-family",
    "font-size",
    "font-weight",
    "line-height",
    "text-align",
    "color",
  ],
  visual: [
    "background",
    "box-shadow",
    "transform",
    "transition",
    "animation",
    "filter",
  ],
};

// 示例CSS代码
const exampleCSS = `.example-container {
  /* 布局相关 */
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  
  /* 盒模型 */
  width: 100%;
  height: auto;
  margin: 20px 10px 15px 10px;
  padding-top: 10px;
  padding-right: 20px;
  padding-bottom: 10px;
  padding-left: 20px;
  
  /* 视觉效果 */
  background-color: #ffffff;
  border: 1px solid #eeeeee;
  border-radius: 4px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  
  /* 文字样式 */
  font-family: Arial, sans-serif;
  font-size: 14px;
  line-height: 1.5;
  color: #333333;
}`;

// 加载示例代码
const loadExample = () => {
  cssInput.value = exampleCSS;
  processCSS();
};

// 清空输入
const clearInput = () => {
  cssInput.value = "";
  cssOutput.value = "";
};

// 复制结果
const copyResult = async () => {
  if (!cssOutput.value) {
    ElMessage.warning("没有可复制的内容");
    return;
  }

  try {
    await navigator.clipboard.writeText(cssOutput.value);
    ElMessage.success("复制成功");
  } catch (err) {
    ElMessage.error("复制失败");
  }
};

// 处理输入变化
const handleInput = () => {
  if (cssInput.value) {
    processCSS();
  } else {
    cssOutput.value = "";
  }
};

// 处理CSS代码
const processCSS = () => {
  if (!cssInput.value) return;

  let result = cssInput.value;

  switch (operationType.value) {
    case "beautify":
      result = beautifyCSS(result);
      break;
    case "purify":
      result = purifyCSS(result);
      break;
    case "organize":
      result = organizeCSS(result);
      break;
    case "optimize":
      result = optimizeCSS(result);
      break;
    case "minify":
      result = minifyCSS(result);
      break;
  }

  cssOutput.value = result;
};

// 美化CSS
const beautifyCSS = (css: string) => {
  // 移除多余的空白和注释
  css = css.trim().replace(/\/\*[\s\S]*?\*\//g, "");

  // 分割规则
  const rules = css.match(/[^}]+}/g) || [];

  // 处理每个规则
  const indentStr =
    beautifyOptions.indent === "tab"
      ? "\t"
      : " ".repeat(Number(beautifyOptions.indent));
  const processedRules = rules.map((rule) => {
    // 分离选择器和属性
    const [selector, ...properties] = rule.split("{");
    const props = properties.join("{").replace("}", "").split(";");

    // 处理属性
    const formattedProps = props
      .map((prop) => prop.trim())
      .filter(Boolean)
      .map((prop) => `${indentStr}${prop};`);

    // 组合规则
    return `${selector.trim()} {\n${formattedProps.join("\n")}\n}`;
  });

  return processedRules.join("\n\n");
};

// 净化CSS
const purifyCSS = (css: string) => {
  return css
    .replace(/\/\*[\s\S]*?\*\//g, "") // 移除注释
    .replace(/[\n\r]/g, "") // 移除换行
    .replace(/\s+/g, " ") // 多个空格转换为单个空格
    .replace(/\s*{\s*/g, "{") // 移除括号周围的空格
    .replace(/\s*}\s*/g, "}") // 移除括号周围的空格
    .replace(/\s*;\s*/g, ";") // 移除分号周围的空格
    .replace(/\s*:\s*/g, ":") // 移除冒号周围的空格
    .trim();
};

// 整理CSS
const organizeCSS = (css: string) => {
  // 首先美化代码
  let beautified = beautifyCSS(css);

  // 分割规则
  const rules = beautified.split("}").filter(Boolean);

  // 处理每个规则
  const processedRules = rules.map((rule) => {
    const [selector, ...properties] = rule.split("{");
    let props = properties
      .join("{")
      .split(";")
      .map((prop) => prop.trim())
      .filter(Boolean);

    // 根据选择的排序方式进行排序
    if (organizeOptions.sortOrder === "alphabetical") {
      props.sort();
    } else {
      // 按属性类型分组排序
      props.sort((a, b) => {
        const getPropGroup = (prop: string) => {
          const propName = prop.split(":")[0].trim();
          for (const [group, properties] of Object.entries(cssPropertyGroups)) {
            if (properties.some((p) => propName.startsWith(p))) {
              return group;
            }
          }
          return "other";
        };

        const groupA = getPropGroup(a);
        const groupB = getPropGroup(b);

        if (groupA === groupB) {
          return a.localeCompare(b);
        }

        const groupOrder = [
          "positioning",
          "display",
          "boxModel",
          "typography",
          "visual",
          "other",
        ];
        return groupOrder.indexOf(groupA) - groupOrder.indexOf(groupB);
      });
    }

    const indentStr = "  ";
    return `${selector.trim()} {\n${props
      .map((prop) => `${indentStr}${prop};`)
      .join("\n")}\n}`;
  });

  return processedRules.join("\n\n");
};

// 优化CSS
const optimizeCSS = (css: string) => {
  return (
    css
      // 优化颜色值
      .replace(/#([0-9a-f])\1([0-9a-f])\2([0-9a-f])\3/gi, "#$1$2$3")
      // 优化0值
      .replace(/(^|[^.])0(?:px|em|rem|%)/g, "$10")
      // 优化小数
      .replace(/(\d+)\.0+(?=[^\d])/g, "$1")
      // 优化margin/padding
      .replace(
        /margin|padding:\s*(\d+\w*)\s+(\d+\w*)\s+(\1)\s+(\2)/g,
        "margin:$1 $2"
      )
      .replace(
        /margin|padding:\s*(\d+\w*)\s+(\d+\w*)\s+(\d+\w*)\s+(\2)/g,
        "margin:$1 $2 $3"
      )
  );
};

// 压缩CSS
const minifyCSS = (css: string) => {
  return css
    .replace(/\/\*[\s\S]*?\*\//g, "") // 移除注释
    .replace(/\s+/g, "") // 移除所有空白
    .replace(/;\}/g, "}") // 移除最后一个分号
    .replace(/:\s+/g, ":") // 移除属性值前的空白
    .replace(/\s*{\s*/g, "{") // 移除括号内的空白
    .replace(/\s*}\s*/g, "}") // 移除括号内的空白
    .replace(/\s*;\s*/g, ";") // 移除分号周围的空白
    .replace(/\s*,\s*/g, ",") // 移除逗号周围的空白
    .trim();
};
</script>

<style scoped>
.container {
  max-width: 1000px;
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

.editor-section,
.result-section {
  margin-bottom: 24px;
}

.editor-header,
.result-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 12px;
}

.editor-header h3,
.result-header h3 {
  font-size: 16px;
  color: var(--el-text-color-primary);
  margin: 0;
}

.actions {
  display: flex;
  gap: 8px;
}

.options-section {
  margin-bottom: 24px;
  padding: 16px;
  background: var(--el-fill-color-light);
  border-radius: 4px;
}

.operation-type {
  margin-bottom: 16px;
}

.operation-options {
  padding-top: 16px;
  border-top: 1px solid var(--el-border-color-light);
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

.usage-guide ul {
  margin: 0;
  padding-left: 20px;
  color: var(--el-text-color-regular);
}

.usage-guide li {
  margin-bottom: 8px;
  line-height: 1.6;
}

.usage-guide strong {
  color: var(--el-text-color-primary);
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

  .operation-type .el-radio-group {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    gap: 8px;
  }
}
</style>
