<template>
  <div class="container">
    <div class="nav-header">
      <el-button link @click="router.push('/')" class="back-btn">
        <el-icon><ArrowLeft /></el-icon>
        返回首页
      </el-button>
    </div>

    <div class="tool-header">
      <el-icon class="header-icon"><Document /></el-icon>
      <div class="header-text">
        <h1>{{ title }}</h1>
        <p class="subtitle">{{ description }}</p>
      </div>
    </div>

    <div class="tool-content">
      <div class="operation-type">
        <el-radio-group v-model="operationType" @change="handleOperation">
          <el-radio-button label="xml2json">XML转JSON</el-radio-button>
          <el-radio-button label="json2xml">JSON转XML</el-radio-button>
        </el-radio-group>
      </div>

      <div class="editor-section">
        <div class="editor-header">
          <h3>输入{{ operationType === "xml2json" ? "XML" : "JSON" }}</h3>
          <div class="actions">
            <el-button size="small" @click="clearInput">
              <el-icon><Delete /></el-icon>
              清空
            </el-button>
            <el-button size="small" @click="loadExample">
              <el-icon><DocumentAdd /></el-icon>
              加载示例
            </el-button>
            <el-button size="small" @click="formatInput">
              <el-icon><Operation /></el-icon>
              格式化
            </el-button>
          </div>
        </div>
        <el-input
          v-model="inputText"
          type="textarea"
          :rows="12"
          :placeholder="inputPlaceholder"
          @input="handleOperation"
        />
      </div>

      <div class="options-section">
        <el-form :model="options" label-width="120px" size="small">
          <el-form-item label="输出格式">
            <el-radio-group v-model="options.format" @change="handleOperation">
              <el-radio label="pretty">格式化</el-radio>
              <el-radio label="compact">压缩</el-radio>
            </el-radio-group>
          </el-form-item>
          <el-form-item v-if="operationType === 'json2xml'" label="根节点名称">
            <el-input
              v-model="options.rootName"
              placeholder="请输入根节点名称"
              @change="handleOperation"
            />
          </el-form-item>
        </el-form>
      </div>

      <div class="result-section">
        <div class="result-header">
          <h3>转换结果</h3>
          <div class="actions">
            <el-button size="small" type="primary" @click="copyResult">
              <el-icon><DocumentCopy /></el-icon>
              复制结果
            </el-button>
          </div>
        </div>
        <el-input
          v-model="outputText"
          type="textarea"
          :rows="12"
          readonly
          :placeholder="outputPlaceholder"
        />
      </div>
    </div>

    <div class="usage-guide">
      <h2>使用说明</h2>
      <ol>
        <li>选择转换方向（XML转JSON或JSON转XML）</li>
        <li>在输入框中粘贴或输入需要转换的内容</li>
        <li>选择输出格式（格式化或压缩）</li>
        <li>如果是JSON转XML，可以指定根节点名称</li>
        <li>点击复制按钮获取转换结果</li>
      </ol>
      <div class="note">
        <el-icon><InfoFilled /></el-icon>
        <span>提示：输入框支持格式化功能，可以帮助您整理代码格式</span>
      </div>
    </div>

    <!-- 添加一个隐藏的input用于复制 -->
    <input ref="copyInput" type="text" class="copy-input" />
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from "vue";
import { useRouter } from "vue-router";
import {
  ArrowLeft,
  Document,
  Delete,
  DocumentAdd,
  DocumentCopy,
  Operation,
  InfoFilled,
} from "@element-plus/icons-vue";
import { ElMessage } from "element-plus";
import toolsData from "~/data/data.json";

const router = useRouter();
const copyInput = ref<HTMLInputElement | null>(null);
const toolInfo = toolsData.tools.find((tool) => tool.id === 29);

// 更新页面标题和描述
const title = toolInfo?.title || "XML转换";
const description = toolInfo?.description || "XML与JSON互转工具";

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

const operationType = ref<"xml2json" | "json2xml">("xml2json");
const inputText = ref("");
const outputText = ref("");
const options = ref({
  format: "pretty",
  rootName: "root",
});

// 计算属性：输入框提示文本
const inputPlaceholder = computed(() => {
  return operationType.value === "xml2json"
    ? "请输入XML文本..."
    : "请输入JSON文本...";
});

// 计算属性：输出框提示文本
const outputPlaceholder = computed(() => {
  return operationType.value === "xml2json"
    ? "JSON结果将显示在这里..."
    : "XML结果将显示在这里...";
});

// XML解析和转换的工具函数
const parseXML = (xmlStr: string) => {
  const parser = new DOMParser();
  const xmlDoc = parser.parseFromString(xmlStr, "text/xml");

  const convertNodeToJson = (node: Node): any => {
    if (node.nodeType === Node.TEXT_NODE) {
      const text = node.textContent?.trim();
      return text === "" ? undefined : text;
    }

    if (node.nodeType === Node.ELEMENT_NODE) {
      const element = node as Element;
      const result: any = {};

      // 处理属性
      if (element.attributes.length > 0) {
        result._attributes = {};
        for (let i = 0; i < element.attributes.length; i++) {
          const attr = element.attributes[i];
          result._attributes[attr.name] = attr.value;
        }
      }

      // 处理子节点
      const children = Array.from(element.childNodes)
        .map((child) => convertNodeToJson(child))
        .filter((child) => child !== undefined);

      if (children.length === 0) {
        return { [element.tagName]: "" };
      } else if (children.length === 1 && typeof children[0] === "string") {
        return { [element.tagName]: children[0] };
      } else {
        const childElements = element.children;
        if (
          childElements.length > 0 &&
          Array.from(childElements).every(
            (el) => el.tagName === childElements[0].tagName
          )
        ) {
          // 处理数组情况
          return { [element.tagName]: children };
        } else {
          const childObj: any = {};
          Array.from(element.children).forEach((child) => {
            const childJson = convertNodeToJson(child);
            Object.assign(childObj, childJson);
          });
          return { [element.tagName]: childObj };
        }
      }
    }
    return undefined;
  };

  return convertNodeToJson(xmlDoc.documentElement);
};

const convertToXML = (obj: any, rootName: string = "root"): string => {
  const createXMLElement = (
    key: string,
    value: any,
    indent: string = ""
  ): string => {
    if (value === null || value === undefined) {
      return `${indent}<${key}/>`;
    }

    if (
      typeof value === "string" ||
      typeof value === "number" ||
      typeof value === "boolean"
    ) {
      return `${indent}<${key}>${value}</${key}>`;
    }

    if (Array.isArray(value)) {
      return value
        .map((item) => createXMLElement(key, item, indent))
        .join("\n");
    }

    if (typeof value === "object") {
      const attrs = value._attributes || {};
      const attrStr = Object.entries(attrs)
        .map(([k, v]) => `${k}="${v}"`)
        .join(" ");

      const children = Object.entries(value)
        .filter(([k]) => k !== "_attributes")
        .map(([k, v]) => createXMLElement(k, v, indent + "  "))
        .join("\n");

      return `${indent}<${key}${
        attrStr ? " " + attrStr : ""
      }>\n${children}\n${indent}</${key}>`;
    }

    return "";
  };

  return `<?xml version="1.0" encoding="UTF-8"?>\n${createXMLElement(
    rootName,
    obj
  )}`;
};

// 处理转换操作
const handleOperation = () => {
  if (!inputText.value) {
    outputText.value = "";
    return;
  }

  try {
    if (operationType.value === "xml2json") {
      const result = parseXML(inputText.value);
      outputText.value =
        options.value.format === "pretty"
          ? JSON.stringify(result, null, 2)
          : JSON.stringify(result);
    } else {
      const jsonData = JSON.parse(inputText.value);
      outputText.value = convertToXML(jsonData, options.value.rootName);
      if (options.value.format === "compact") {
        outputText.value = outputText.value
          .replace(/\s+/g, " ")
          .replace(/>\s+</g, "><")
          .trim();
      }
    }
  } catch (error) {
    console.error(error);
    ElMessage.error("转换失败，请检查输入格式是否正确");
  }
};

// 格式化输入
const formatInput = () => {
  if (!inputText.value) return;

  try {
    if (operationType.value === "xml2json") {
      const result = parseXML(inputText.value);
      inputText.value = convertToXML(result);
    } else {
      const jsonData = JSON.parse(inputText.value);
      inputText.value = JSON.stringify(jsonData, null, 2);
    }
  } catch (error) {
    console.error(error);
    ElMessage.error("格式化失败，请检查输入格式是否正确");
  }
};

// 清空输入
const clearInput = () => {
  inputText.value = "";
  outputText.value = "";
};

// 加载示例
const loadExample = () => {
  if (operationType.value === "xml2json") {
    inputText.value = `<?xml version="1.0" encoding="UTF-8"?>
<root>
  <person>
    <name>张三</name>
    <age>25</age>
    <hobbies>
      <hobby>读书</hobby>
      <hobby>旅游</hobby>
    </hobbies>
  </person>
</root>`;
  } else {
    inputText.value = `{
  "person": {
    "name": "张三",
    "age": 25,
    "hobbies": {
      "hobby": ["读书", "旅游"]
    }
  }
}`;
  }
  handleOperation();
};

// 复制结果
const copyResult = () => {
  if (!outputText.value) return;
  if (!copyInput.value) return;

  copyInput.value.value = outputText.value;
  copyInput.value.select();
  try {
    document.execCommand("copy");
    ElMessage.success("复制成功");
  } catch (err) {
    ElMessage.error("复制失败，请手动复制");
  }
  copyInput.value.value = "";
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

.operation-type {
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
  margin: 24px 0;
  padding: 16px;
  background: var(--el-fill-color-light);
  border-radius: 4px;
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

.copy-input {
  position: absolute;
  top: -9999px;
  left: -9999px;
  opacity: 0;
  pointer-events: none;
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

  .editor-header,
  .result-header {
    flex-direction: column;
    align-items: flex-start;
    gap: 8px;
  }

  .actions {
    width: 100%;
    justify-content: flex-end;
  }
}
</style>
