<template>
  <div class="container">
    <div class="nav-header">
      <el-button link @click="router.push('/')" class="back-btn">
        <el-icon><ArrowLeft /></el-icon>
        返回首页
      </el-button>
    </div>

    <div class="tool-header">
      <el-icon class="header-icon"><Monitor /></el-icon>
      <div class="header-text">
        <h1>{{ title }}</h1>
        <p class="subtitle">{{ description }}</p>
      </div>
    </div>

    <div class="tool-content">
      <div class="settings-section">
        <el-form :model="settings" label-width="120px">
          <el-form-item label="数据库类型">
            <el-select v-model="settings.language" @change="handleFormat">
              <el-option label="标准SQL" value="sql" />
              <el-option label="MySQL" value="mysql" />
              <el-option label="PostgreSQL" value="postgresql" />
              <el-option label="SQL Server" value="tsql" />
              <el-option label="Oracle" value="plsql" />
            </el-select>
          </el-form-item>
          <el-form-item label="缩进">
            <el-input-number
              v-model="settings.indent"
              :min="2"
              :max="8"
              :step="2"
              @change="handleFormat"
            />
          </el-form-item>
          <el-form-item label="大写关键字">
            <el-switch v-model="settings.uppercase" @change="handleFormat" />
          </el-form-item>
        </el-form>
      </div>

      <div class="editor-section">
        <div class="editor-container">
          <div class="editor-header">
            <h3>输入SQL</h3>
            <div class="editor-actions">
              <el-dropdown @command="handleExample">
                <el-button size="small">
                  <el-icon><Document /></el-icon>
                  插入示例
                  <el-icon class="el-icon--right"><ArrowDown /></el-icon>
                </el-button>
                <template #dropdown>
                  <el-dropdown-menu>
                    <el-dropdown-item
                      v-for="example in SQL_EXAMPLES"
                      :key="example.label"
                      :command="example.value"
                    >
                      {{ example.label }}
                    </el-dropdown-item>
                  </el-dropdown-menu>
                </template>
              </el-dropdown>
              <el-button @click="clearInput" size="small">
                <el-icon><Delete /></el-icon>
                清空
              </el-button>
              <el-button @click="handleFormat" type="primary" size="small">
                <el-icon><Operation /></el-icon>
                格式化
              </el-button>
            </div>
          </div>
          <el-input
            v-model="sqlInput"
            type="textarea"
            :rows="10"
            placeholder="请输入要格式化的SQL语句..."
            @input="handleInputChange"
          />
        </div>

        <div class="editor-container" v-if="formattedSql">
          <div class="editor-header">
            <h3>格式化结果</h3>
            <div class="editor-actions">
              <el-button @click="copyToClipboard" size="small">
                <el-icon><CopyDocument /></el-icon>
                复制
              </el-button>
            </div>
          </div>
          <el-input
            v-model="formattedSql"
            type="textarea"
            :rows="10"
            readonly
          />
        </div>
      </div>
    </div>

    <div class="usage-guide">
      <h2>使用说明</h2>
      <ol>
        <li>在输入框中粘贴或输入需要格式化的SQL语句</li>
        <li>
          选择合适的格式化选项：
          <ul>
            <li>数据库类型：选择对应的数据库语法</li>
            <li>缩进：设置缩进空格数（2-8个空格）</li>
            <li>大写关键字：是否将SQL关键字转换为大写</li>
          </ul>
        </li>
        <li>点击"格式化"按钮或等待自动格式化</li>
        <li>使用"复制"按钮复制格式化后的SQL</li>
      </ol>
      <div class="note">
        <el-icon><InfoFilled /></el-icon>
        <span
          >提示：工具会自动检测并修复常见的SQL语法格式问题，但不会改变SQL语句的功能</span
        >
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive, watch } from "vue";
import { useRouter } from "vue-router";
import {
  ArrowLeft,
  Monitor,
  Delete,
  Operation,
  CopyDocument,
  InfoFilled,
  Document,
  ArrowDown,
} from "@element-plus/icons-vue";
import { ElMessage } from "element-plus";
import toolsData from "~/data/data.json";

const router = useRouter();
const toolInfo = toolsData.tools.find((tool) => tool.id === 35);

// 更新页面标题和描述
const title = toolInfo?.title || "SQL格式化";
const description = toolInfo?.description || "SQL语句格式化工具";

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

// SQL输入和输出
const sqlInput = ref("");
const formattedSql = ref("");

// 格式化设置
const settings = reactive({
  language: "sql",
  indent: 2,
  uppercase: true,
});

// 防抖定时器
let formatTimer: NodeJS.Timeout | null = null;

// 监听输入变化
const handleInputChange = () => {
  if (formatTimer) {
    clearTimeout(formatTimer);
  }
  formatTimer = setTimeout(() => {
    handleFormat();
  }, 1000);
};

// SQL关键字列表
const SQL_KEYWORDS = [
  "SELECT",
  "FROM",
  "WHERE",
  "INSERT",
  "UPDATE",
  "DELETE",
  "CREATE",
  "DROP",
  "ALTER",
  "TABLE",
  "INTO",
  "VALUES",
  "SET",
  "AND",
  "OR",
  "NOT",
  "NULL",
  "IS",
  "IN",
  "BETWEEN",
  "LIKE",
  "ORDER BY",
  "GROUP BY",
  "HAVING",
  "JOIN",
  "LEFT",
  "RIGHT",
  "INNER",
  "OUTER",
  "ON",
  "AS",
  "DISTINCT",
  "COUNT",
  "SUM",
  "AVG",
  "MAX",
  "MIN",
  "UNION",
  "ALL",
  "CASE",
  "WHEN",
  "THEN",
  "ELSE",
  "END",
  "IF",
  "EXISTS",
  "LIMIT",
  "OFFSET",
];

// SQL示例列表
const SQL_EXAMPLES = [
  {
    label: "简单查询",
    value: `select id, name, age from users where age > 18 order by name`,
  },
  {
    label: "多表联接",
    value: `select u.name, o.order_no, o.create_time 
from users u 
left join orders o on u.id = o.user_id 
where o.status = 'completed' 
and o.create_time >= '2024-01-01'`,
  },
  {
    label: "分组统计",
    value: `select 
    department, 
    count(*) as employee_count,
    avg(salary) as avg_salary,
    max(salary) as max_salary
from employees 
where status = 'active' 
group by department 
having count(*) > 5 
order by avg_salary desc`,
  },
  {
    label: "插入数据",
    value: `insert into users (name, email, age, create_time) 
values 
('张三', 'zhangsan@example.com', 25, now()),
('李四', 'lisi@example.com', 28, now())`,
  },
  {
    label: "更新数据",
    value: `update products 
set 
    price = price * 1.1,
    update_time = now(),
    modified_by = 'admin'
where category = 'electronics' 
and status = 'active'`,
  },
  {
    label: "创建表",
    value: `create table users (
    id bigint primary key auto_increment,
    name varchar(50) not null comment '用户名',
    email varchar(100) unique comment '邮箱',
    age int check (age > 0 and age < 150),
    status varchar(20) default 'active',
    create_time datetime not null,
    update_time datetime,
    index idx_name (name),
    index idx_email (email)
) comment '用户表'`,
  },
];

// 格式化SQL
const handleFormat = () => {
  if (!sqlInput.value.trim()) {
    formattedSql.value = "";
    return;
  }

  try {
    let sql = sqlInput.value.trim();

    // 1. 统一空白字符
    sql = sql.replace(/\s+/g, " ");

    // 2. 处理括号
    sql = sql.replace(/\(/g, " ( ").replace(/\)/g, " ) ");

    // 3. 处理逗号
    sql = sql.replace(/,/g, ", ");

    // 4. 分割为词组
    let tokens = sql.split(" ").filter((token) => token.trim());

    // 5. 大写关键字
    if (settings.uppercase) {
      tokens = tokens.map((token) => {
        if (SQL_KEYWORDS.includes(token.toUpperCase())) {
          return token.toUpperCase();
        }
        return token;
      });
    }

    // 6. 添加换行和缩进
    let result = "";
    let indentLevel = 0;
    let indent = " ".repeat(settings.indent);

    for (let i = 0; i < tokens.length; i++) {
      let token = tokens[i];
      let prevToken = tokens[i - 1] || "";
      let nextToken = tokens[i + 1] || "";

      // 减少缩进级别
      if (token === ")") {
        indentLevel = Math.max(0, indentLevel - 1);
      }

      // 添加换行
      if (i > 0 && shouldAddNewline(token, prevToken)) {
        result += "\n" + indent.repeat(indentLevel);
      } else if (i > 0) {
        result += " ";
      }

      // 添加当前token
      result += token;

      // 增加缩进级别
      if (token === "(") {
        indentLevel++;
      }

      // 特殊处理某些关键字后的换行
      if (shouldAddNewlineAfter(token, nextToken)) {
        result += "\n" + indent.repeat(indentLevel);
      }
    }

    formattedSql.value = result;
  } catch (error) {
    ElMessage.error("SQL语句格式化失败，请检查语法是否正确");
  }
};

// 判断是否需要在token前添加换行
const shouldAddNewline = (token: string, prevToken: string) => {
  const newlineKeywords = [
    "SELECT",
    "FROM",
    "WHERE",
    "ORDER BY",
    "GROUP BY",
    "HAVING",
    "UNION",
    "INSERT",
    "UPDATE",
    "DELETE",
    "SET",
  ];

  if (newlineKeywords.includes(token.toUpperCase())) {
    return true;
  }

  if (token.toUpperCase() === "AND" || token.toUpperCase() === "OR") {
    return true;
  }

  return false;
};

// 判断是否需要在token后添加换行
const shouldAddNewlineAfter = (token: string, nextToken: string) => {
  if (token === ",") {
    return true;
  }
  return false;
};

// 清空输入
const clearInput = () => {
  sqlInput.value = "";
  formattedSql.value = "";
};

// 复制到剪贴板
const copyToClipboard = async () => {
  if (!formattedSql.value) return;

  try {
    await navigator.clipboard.writeText(formattedSql.value);
    ElMessage.success("已复制到剪贴板");
  } catch (error) {
    ElMessage.error("复制失败，请手动复制");
  }
};

// 处理示例SQL
const handleExample = (sql: string) => {
  sqlInput.value = sql;
  handleFormat();
};

// 监听输入变化自动格式化
watch(sqlInput, () => {
  handleInputChange();
});
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

.settings-section {
  margin-bottom: 24px;
  padding-bottom: 24px;
  border-bottom: 1px solid var(--el-border-color-light);
}

.editor-section {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 24px;
}

.editor-container {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.editor-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.editor-header h3 {
  font-size: 16px;
  margin: 0;
  color: var(--el-text-color-primary);
}

.editor-actions {
  display: flex;
  gap: 8px;
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

  .editor-section {
    grid-template-columns: 1fr;
  }
}
</style>
