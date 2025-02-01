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
        <h1>JSON转换</h1>
        <p class="subtitle">
          在线JSON转换工具，支持转换为YAML、Java类、PHP类、Go结构体等
        </p>
      </div>
    </div>

    <div class="tool-content">
      <div class="editor-section">
        <div class="editor-header">
          <h3>输入JSON</h3>
          <div class="actions">
            <el-button size="small" @click="formatJSON">
              <el-icon><Operation /></el-icon>
              格式化
            </el-button>
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
          v-model="jsonInput"
          type="textarea"
          :rows="10"
          placeholder="请输入JSON代码..."
          @input="handleInput"
        />
      </div>

      <div class="options-section">
        <div class="operation-type">
          <el-radio-group v-model="convertType" @change="convertJSON">
            <el-radio-button label="yaml">YAML</el-radio-button>
            <el-radio-button label="java">Java类</el-radio-button>
            <el-radio-button label="php5">PHP5类</el-radio-button>
            <el-radio-button label="php7">PHP7类</el-radio-button>
            <el-radio-button label="go">Go结构体</el-radio-button>
          </el-radio-group>
        </div>

        <div class="operation-options">
          <el-form :model="convertOptions" label-width="120px" size="small">
            <el-form-item label="类名" v-if="convertType !== 'yaml'">
              <el-input
                v-model="convertOptions.className"
                placeholder="请输入类名"
                @input="convertJSON"
              />
            </el-form-item>
            <el-form-item
              label="包名"
              v-if="convertType === 'java' || convertType === 'go'"
            >
              <el-input
                v-model="convertOptions.packageName"
                placeholder="请输入包名"
                @input="convertJSON"
              />
            </el-form-item>
            <el-form-item
              label="访问修饰符"
              v-if="convertType === 'java' || convertType.startsWith('php')"
            >
              <el-radio-group
                v-model="convertOptions.accessModifier"
                @change="convertJSON"
              >
                <el-radio label="public">public</el-radio>
                <el-radio label="private">private</el-radio>
                <el-radio label="protected">protected</el-radio>
              </el-radio-group>
            </el-form-item>
            <el-form-item label="Getter/Setter" v-if="convertType === 'java'">
              <el-checkbox
                v-model="convertOptions.generateGetterSetter"
                @change="convertJSON"
              >
                生成Getter/Setter方法
              </el-checkbox>
            </el-form-item>
          </el-form>
        </div>
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
          v-model="convertResult"
          type="textarea"
          :rows="15"
          readonly
          placeholder="转换后的代码将显示在这里..."
        />
      </div>
    </div>

    <div class="usage-guide">
      <h2>功能说明</h2>
      <ul>
        <li>
          <strong>YAML转换：</strong>将JSON转换为YAML格式，保持数据结构不变
        </li>
        <li>
          <strong>Java类：</strong>生成对应的Java
          POJO类，支持生成Getter/Setter方法
        </li>
        <li><strong>PHP5类：</strong>生成PHP5风格的类定义，包含属性和方法</li>
        <li><strong>PHP7类：</strong>生成PHP7风格的类定义，支持类型声明</li>
        <li><strong>Go结构体：</strong>生成Go语言的结构体定义，包含json标签</li>
      </ul>
      <div class="note">
        <el-icon><InfoFilled /></el-icon>
        <span>提示：所有转换都在本地进行，不会上传到服务器</span>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive } from "vue";
import { useRouter } from "vue-router";
import {
  ArrowLeft,
  Document,
  Operation,
  Delete,
  DocumentAdd,
  DocumentCopy,
  InfoFilled,
} from "@element-plus/icons-vue";
import { ElMessage } from "element-plus";
import yaml from "js-yaml";
import toolsData from "~/data/data.json";

const router = useRouter();

// 从data.json导入工具信息
const toolInfo = toolsData.tools.find((tool) => tool.id === 25);

// 更新页面标题和描述
const title = toolInfo?.title || "JSON转换";
const description = toolInfo?.description || "在线JSON转换工具";

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

const jsonInput = ref("");
const convertResult = ref("");
const convertType = ref("yaml");

// 转换选项
const convertOptions = reactive({
  className: "MyClass",
  packageName: "com.example",
  accessModifier: "public",
  generateGetterSetter: true,
});

// 示例JSON
const exampleJSON = {
  id: 1,
  name: "John Doe",
  age: 30,
  email: "john@example.com",
  address: {
    street: "123 Main St",
    city: "New York",
    country: "USA",
    zipCode: "10001",
  },
  tags: ["developer", "designer"],
  active: true,
  salary: 75000.5,
  createdAt: "2024-01-01T00:00:00Z",
};

// 加载示例
const loadExample = () => {
  jsonInput.value = JSON.stringify(exampleJSON, null, 2);
  convertJSON();
};

// 清空输入
const clearInput = () => {
  jsonInput.value = "";
  convertResult.value = "";
};

// 格式化JSON
const formatJSON = () => {
  try {
    const obj = JSON.parse(jsonInput.value);
    jsonInput.value = JSON.stringify(obj, null, 2);
    convertJSON();
  } catch (err) {
    ElMessage.error("JSON格式错误");
  }
};

// 复制结果
const copyResult = async () => {
  if (!convertResult.value) {
    ElMessage.warning("没有可复制的内容");
    return;
  }

  try {
    // 创建临时文本区域
    const textArea = document.createElement("textarea");
    textArea.value = convertResult.value;
    document.body.appendChild(textArea);

    // 选择文本
    textArea.select();
    textArea.setSelectionRange(0, 99999); // 用于移动设备

    // 尝试使用新API
    try {
      await navigator.clipboard.writeText(convertResult.value);
      ElMessage.success("复制成功");
    } catch {
      // 如果新API失败，使用传统方法
      document.execCommand("copy");
      ElMessage.success("复制成功");
    }

    // 清理
    document.body.removeChild(textArea);
  } catch (err) {
    ElMessage.error("复制失败");
    console.error("复制失败:", err);
  }
};

// 处理输入变化
const handleInput = () => {
  if (jsonInput.value) {
    convertJSON();
  } else {
    convertResult.value = "";
  }
};

// 转换JSON
const convertJSON = () => {
  if (!jsonInput.value) return;

  try {
    const jsonObj = JSON.parse(jsonInput.value);

    switch (convertType.value) {
      case "yaml":
        convertResult.value = yaml.dump(jsonObj);
        break;
      case "java":
        convertResult.value = generateJavaClass(jsonObj);
        break;
      case "php5":
        convertResult.value = generatePHP5Class(jsonObj);
        break;
      case "php7":
        convertResult.value = generatePHP7Class(jsonObj);
        break;
      case "go":
        convertResult.value = generateGoStruct(jsonObj);
        break;
    }
  } catch (err) {
    ElMessage.error("JSON格式错误");
    convertResult.value = "";
  }
};

interface TypeInfo {
  java: string;
  php5: string;
  php7: string;
  go: string;
  isObject: boolean;
  value?: any;
}

// 获取JSON值的类型
const getTypeInfo = (value: any, key: string = ""): TypeInfo => {
  if (Array.isArray(value)) {
    const itemType: TypeInfo =
      value.length > 0
        ? getTypeInfo(value[0], key)
        : {
            java: "Object",
            php5: "mixed",
            php7: "mixed",
            go: "interface{}",
            isObject: false,
          };
    return {
      java: `List<${itemType.java}>`,
      php5: "array",
      php7: "array",
      go: `[]${itemType.go}`,
      isObject: false,
    };
  }

  switch (typeof value) {
    case "string":
      return {
        java: "String",
        php5: "string",
        php7: "string",
        go: "string",
        isObject: false,
      };
    case "number":
      if (Number.isInteger(value)) {
        return {
          java: "Integer",
          php5: "int",
          php7: "int",
          go: "int",
          isObject: false,
        };
      }
      return {
        java: "Double",
        php5: "float",
        php7: "float",
        go: "float64",
        isObject: false,
      };
    case "boolean":
      return {
        java: "Boolean",
        php5: "bool",
        php7: "bool",
        go: "bool",
        isObject: false,
      };
    case "object":
      if (value === null) {
        return {
          java: "Object",
          php5: "mixed",
          php7: "mixed",
          go: "interface{}",
          isObject: false,
        };
      }
      const typeName = key.charAt(0).toUpperCase() + key.slice(1);
      return {
        java: convertOptions.className + typeName,
        php5: "array",
        php7: "array",
        go: typeName,
        isObject: true,
        value,
      };
    default:
      return {
        java: "Object",
        php5: "mixed",
        php7: "mixed",
        go: "interface{}",
        isObject: false,
      };
  }
};

// 生成Java类
const generateJavaClass = (
  json: any,
  className: string = convertOptions.className
) => {
  let result = "";
  const nestedClasses = new Map<string, string>();

  const generateClass = (obj: any, name: string) => {
    let classDef = `${convertOptions.accessModifier} class ${name} {\n`;

    // 属性
    for (const [key, value] of Object.entries(obj)) {
      const typeInfo = getTypeInfo(value, key);
      if (typeInfo.isObject) {
        const nestedClassName = key.charAt(0).toUpperCase() + key.slice(1);
        classDef += `    private ${nestedClassName} ${key};\n`;

        if (!nestedClasses.has(nestedClassName)) {
          nestedClasses.set(
            nestedClassName,
            generateClass(value, nestedClassName)
          );
        }
      } else {
        classDef += `    private ${typeInfo.java} ${key};\n`;
      }
    }

    // Getter/Setter
    if (convertOptions.generateGetterSetter) {
      classDef += "\n";
      for (const [key, value] of Object.entries(obj)) {
        const typeInfo = getTypeInfo(value, key);
        const capitalizedKey = key.charAt(0).toUpperCase() + key.slice(1);
        const type = typeInfo.isObject ? capitalizedKey : typeInfo.java;

        // Getter
        classDef += `    public ${type} get${capitalizedKey}() {\n`;
        classDef += `        return ${key};\n`;
        classDef += `    }\n\n`;

        // Setter
        classDef += `    public void set${capitalizedKey}(${type} ${key}) {\n`;
        classDef += `        this.${key} = ${key};\n`;
        classDef += `    }\n\n`;
      }
    }

    classDef += "}\n\n";
    return classDef;
  };

  if (convertOptions.packageName) {
    result += `package ${convertOptions.packageName};\n\n`;
  }

  result += `import java.util.List;\n\n`;

  // 先生成主类
  const mainClass = generateClass(json, className);

  // 生成所有嵌套类
  for (const [_, classDef] of nestedClasses) {
    result += classDef;
  }

  // 添加主类
  result += mainClass;

  return result.trim();
};

// 生成PHP5类
const generatePHP5Class = (
  json: any,
  className: string = convertOptions.className
) => {
  let result = "<?php\n\n";
  const nestedClasses = new Map<string, string>();

  const generateClass = (obj: any, name: string) => {
    let classDef = `${convertOptions.accessModifier} class ${name} {\n`;

    // 属性
    for (const [key, value] of Object.entries(obj)) {
      const typeInfo = getTypeInfo(value, key);
      if (typeInfo.isObject) {
        const nestedClassName = key.charAt(0).toUpperCase() + key.slice(1);
        classDef += `    /** @var ${nestedClassName} */\n`;
        classDef += `    ${convertOptions.accessModifier} $${key};\n`;

        if (!nestedClasses.has(nestedClassName)) {
          generateClass(value, nestedClassName);
        }
      } else if (Array.isArray(value)) {
        classDef += `    /** @var ${typeInfo.php5}<${
          value.length > 0 ? getTypeInfo(value[0], key).php5 : "mixed"
        }> */\n`;
        classDef += `    ${convertOptions.accessModifier} $${key};\n`;
      } else {
        classDef += `    /** @var ${typeInfo.php5} */\n`;
        classDef += `    ${convertOptions.accessModifier} $${key};\n`;
      }
    }

    // Getter/Setter
    classDef += "\n";
    for (const [key, value] of Object.entries(obj)) {
      const capitalizedKey = key.charAt(0).toUpperCase() + key.slice(1);

      // Getter
      classDef += `    public function get${capitalizedKey}() {\n`;
      classDef += `        return $this->${key};\n`;
      classDef += `    }\n\n`;

      // Setter
      classDef += `    public function set${capitalizedKey}($value) {\n`;
      classDef += `        $this->${key} = $value;\n`;
      classDef += `        return $this;\n`;
      classDef += `    }\n\n`;
    }

    classDef += "}\n\n";
    nestedClasses.set(name, classDef);
    return classDef;
  };

  // 生成所有类
  generateClass(json, className);

  // 按照依赖顺序添加类定义
  const addedClasses = new Set<string>();
  const addClass = (name: string) => {
    if (addedClasses.has(name)) return;

    const classDef = nestedClasses.get(name);
    if (classDef) {
      result += classDef;
      addedClasses.add(name);
    }
  };

  // 先添加最深层的嵌套类
  const processObject = (obj: any) => {
    for (const [key, value] of Object.entries(obj)) {
      if (typeof value === "object" && value !== null) {
        if (!Array.isArray(value)) {
          const nestedClassName = key.charAt(0).toUpperCase() + key.slice(1);
          processObject(value);
          addClass(nestedClassName);
        }
      }
    }
  };

  processObject(json);
  addClass(className);

  return result.trim();
};

// 生成PHP7类
const generatePHP7Class = (
  json: any,
  className: string = convertOptions.className
) => {
  let result = "<?php\n\n";
  result += `declare(strict_types=1);\n\n`;
  const nestedClasses = new Map<string, string>();

  const generateClass = (obj: any, name: string) => {
    let classDef = `${convertOptions.accessModifier} class ${name} {\n`;

    // 属性
    for (const [key, value] of Object.entries(obj)) {
      const typeInfo = getTypeInfo(value, key);
      if (typeInfo.isObject) {
        const nestedClassName = key.charAt(0).toUpperCase() + key.slice(1);
        classDef += `    ${convertOptions.accessModifier} ${nestedClassName} $${key};\n`;

        if (!nestedClasses.has(nestedClassName)) {
          generateClass(value, nestedClassName);
        }
      } else if (Array.isArray(value)) {
        classDef += `    ${convertOptions.accessModifier} array $${key};\n`;
      } else {
        classDef += `    ${convertOptions.accessModifier} ${typeInfo.php7} $${key};\n`;
      }
    }

    // Getter/Setter
    classDef += "\n";
    for (const [key, value] of Object.entries(obj)) {
      const typeInfo = getTypeInfo(value, key);
      const capitalizedKey = key.charAt(0).toUpperCase() + key.slice(1);
      const type = typeInfo.isObject
        ? key.charAt(0).toUpperCase() + key.slice(1)
        : Array.isArray(value)
        ? "array"
        : typeInfo.php7;

      // Getter
      classDef += `    public function get${capitalizedKey}(): ${type} {\n`;
      classDef += `        return $this->${key};\n`;
      classDef += `    }\n\n`;

      // Setter
      classDef += `    public function set${capitalizedKey}(${type} $value): self {\n`;
      classDef += `        $this->${key} = $value;\n`;
      classDef += `        return $this;\n`;
      classDef += `    }\n\n`;
    }

    classDef += "}\n\n";
    nestedClasses.set(name, classDef);
    return classDef;
  };

  // 生成所有类
  generateClass(json, className);

  // 按照依赖顺序添加类定义
  const addedClasses = new Set<string>();
  const addClass = (name: string) => {
    if (addedClasses.has(name)) return;

    const classDef = nestedClasses.get(name);
    if (classDef) {
      result += classDef;
      addedClasses.add(name);
    }
  };

  // 先添加最深层的嵌套类
  const processObject = (obj: any) => {
    for (const [key, value] of Object.entries(obj)) {
      if (typeof value === "object" && value !== null) {
        if (!Array.isArray(value)) {
          const nestedClassName = key.charAt(0).toUpperCase() + key.slice(1);
          processObject(value);
          addClass(nestedClassName);
        }
      }
    }
  };

  processObject(json);
  addClass(className);

  return result.trim();
};

// 生成Go结构体
const generateGoStruct = (
  json: any,
  structName: string = convertOptions.className
) => {
  let result = "";
  const nestedStructs = new Set<string>();
  const generatedStructs = new Map<string, string>();

  const generateStruct = (obj: any, name: string) => {
    let structDef = `type ${name} struct {\n`;

    for (const [key, value] of Object.entries(obj)) {
      const typeInfo = getTypeInfo(value, key);
      const capitalizedKey = key.charAt(0).toUpperCase() + key.slice(1);

      if (typeInfo.isObject) {
        const nestedStructName = capitalizedKey;
        nestedStructs.add(nestedStructName);
        structDef += `    ${capitalizedKey} ${nestedStructName} \`json:"${key}"\`\n`;

        if (!generatedStructs.has(nestedStructName)) {
          generatedStructs.set(
            nestedStructName,
            generateStruct(typeInfo.value, nestedStructName)
          );
        }
      } else {
        structDef += `    ${capitalizedKey} ${typeInfo.go} \`json:"${key}"\`\n`;
      }
    }

    structDef += "}\n\n";
    return structDef;
  };

  if (convertOptions.packageName) {
    result += `package ${convertOptions.packageName}\n\n`;
  }

  const mainStruct = generateStruct(json, structName);

  // 添加所有嵌套结构体的定义
  for (const [name, def] of generatedStructs) {
    result += def;
  }

  // 添加主结构体的定义
  result += mainStruct;

  return result.trim();
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
