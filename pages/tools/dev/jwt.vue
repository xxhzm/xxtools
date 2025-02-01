<template>
  <div class="container">
    <div class="nav-header">
      <el-button link @click="router.push('/')" class="back-btn">
        <el-icon><ArrowLeft /></el-icon>
        返回首页
      </el-button>
    </div>

    <div class="tool-header">
      <el-icon class="header-icon"><Key /></el-icon>
      <div class="header-text">
        <h1>{{ title }}</h1>
        <p class="subtitle">{{ description }}</p>
      </div>
    </div>

    <div class="tool-content">
      <div class="jwt-section">
        <div class="input-section">
          <el-form :model="form" label-width="100px">
            <el-form-item label="JWT令牌">
              <div class="jwt-input">
                <el-input
                  v-model="form.token"
                  type="textarea"
                  :rows="6"
                  :autosize="{ minRows: 6, maxRows: 10 }"
                  style="width: 100%"
                  placeholder="请输入JWT令牌"
                  @input="parseToken"
                >
                </el-input>
                <div class="input-buttons">
                  <el-button @click="insertExample" type="primary">
                    <el-icon><Document /></el-icon>
                    示例
                  </el-button>
                  <el-button @click="clearToken">
                    <el-icon><Delete /></el-icon>
                    清空
                  </el-button>
                </div>
              </div>
            </el-form-item>
          </el-form>
        </div>

        <div class="result-section">
          <div v-if="!form.token" class="empty-state">
            <el-icon><Document /></el-icon>
            <p>请输入JWT令牌进行解析</p>
          </div>
          <template v-else>
            <div class="jwt-part header">
              <div class="part-header">
                <h3>Header</h3>
                <div class="part-label">头部</div>
              </div>
              <pre class="json-content">{{
                formatJson(parsedToken.header)
              }}</pre>
            </div>

            <div class="jwt-part payload">
              <div class="part-header">
                <h3>Payload</h3>
                <div class="part-label">负载</div>
              </div>
              <pre class="json-content">{{
                formatJson(parsedToken.payload)
              }}</pre>
              <div class="time-info" v-if="hasTimeInfo">
                <div class="time-item" v-if="parsedToken.payload.iat">
                  <span class="label">签发时间：</span>
                  <span class="value">{{
                    formatUnixTime(parsedToken.payload.iat)
                  }}</span>
                </div>
                <div class="time-item" v-if="parsedToken.payload.exp">
                  <span class="label">过期时间：</span>
                  <span class="value">{{
                    formatUnixTime(parsedToken.payload.exp)
                  }}</span>
                </div>
                <div class="time-item" v-if="parsedToken.payload.nbf">
                  <span class="label">生效时间：</span>
                  <span class="value">{{
                    formatUnixTime(parsedToken.payload.nbf)
                  }}</span>
                </div>
              </div>
            </div>

            <div class="jwt-part signature">
              <div class="part-header">
                <h3>Signature</h3>
                <div class="part-label">签名</div>
              </div>
              <div class="signature-content">{{ parsedToken.signature }}</div>
            </div>
          </template>
        </div>
      </div>
    </div>

    <div class="usage-guide">
      <h2>使用说明</h2>
      <ol>
        <li>
          JWT格式说明：
          <ul>
            <li>JWT由三部分组成：Header.Payload.Signature</li>
            <li>每部分都是Base64Url编码的字符串，中间用点号分隔</li>
          </ul>
        </li>
        <li>
          常见字段说明：
          <ul>
            <li>iss (Issuer)：签发者</li>
            <li>sub (Subject)：主题</li>
            <li>aud (Audience)：受众</li>
            <li>exp (Expiration Time)：过期时间</li>
            <li>nbf (Not Before)：生效时间</li>
            <li>iat (Issued At)：签发时间</li>
            <li>jti (JWT ID)：编号</li>
          </ul>
        </li>
        <li>工具会自动解析JWT的三个部分并格式化显示</li>
        <li>
          对于时间相关字段（exp、nbf、iat），会自动转换为可读的日期时间格式
        </li>
      </ol>
      <div class="note">
        <el-icon><InfoFilled /></el-icon>
        <span>注意：本工具仅用于解析JWT内容，不会验证签名的有效性</span>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive, computed } from "vue";
import { useRouter } from "vue-router";
import {
  ArrowLeft,
  Key,
  Delete,
  Document,
  InfoFilled,
} from "@element-plus/icons-vue";
import toolsData from "~/data/data.json";
import dayjs from "dayjs";

const router = useRouter();
const toolInfo = toolsData.tools.find((tool) => tool.id === 39);

// 更新页面标题和描述
const title = toolInfo?.title || "JWT解析";
const description = toolInfo?.description || "JWT令牌解析工具";

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

interface JWTPayload {
  iat?: number;
  exp?: number;
  nbf?: number;
  [key: string]: any;
}

interface ParsedToken {
  header: Record<string, any>;
  payload: JWTPayload;
  signature: string;
}

// 表单数据
const form = reactive({
  token: "",
});

// 解析后的令牌数据
const parsedToken = reactive<ParsedToken>({
  header: {},
  payload: {},
  signature: "",
});

// 是否包含时间信息
const hasTimeInfo = computed(() => {
  const payload = parsedToken.payload as any;
  return payload.iat || payload.exp || payload.nbf;
});

// Base64Url解码
const base64UrlDecode = (input: string): string => {
  // 将Base64Url转换为标准Base64
  const base64 = input
    .replace(/-/g, "+")
    .replace(/_/g, "/")
    .padEnd(input.length + ((4 - (input.length % 4)) % 4), "=");

  try {
    // 使用TextDecoder解码UTF-8字符
    const binaryStr = atob(base64);
    const bytes = new Uint8Array(binaryStr.length);
    for (let i = 0; i < binaryStr.length; i++) {
      bytes[i] = binaryStr.charCodeAt(i);
    }
    return new TextDecoder("utf-8").decode(bytes);
  } catch (error) {
    console.error("Base64 decode error:", error);
    throw error;
  }
};

// 解析JWT令牌
const parseToken = () => {
  try {
    if (!form.token) {
      resetParsedToken();
      return;
    }

    const parts = form.token.split(".");
    if (parts.length !== 3) {
      throw new Error("Invalid JWT format");
    }

    // 解析Header
    parsedToken.header = JSON.parse(base64UrlDecode(parts[0]));

    // 解析Payload
    parsedToken.payload = JSON.parse(base64UrlDecode(parts[1]));

    // 保存Signature
    parsedToken.signature = parts[2];
  } catch (error) {
    console.error("Error parsing JWT:", error);
    resetParsedToken();
  }
};

// 重置解析结果
const resetParsedToken = () => {
  parsedToken.header = {};
  parsedToken.payload = {};
  parsedToken.signature = "";
};

// 清空输入
const clearToken = () => {
  form.token = "";
  resetParsedToken();
};

// 格式化JSON显示
const formatJson = (obj: any) => {
  try {
    return JSON.stringify(obj, null, 2);
  } catch (error) {
    return "{}";
  }
};

// 格式化Unix时间戳
const formatUnixTime = (timestamp: number) => {
  return dayjs.unix(timestamp).format("YYYY-MM-DD HH:mm:ss");
};

// 示例JWT令牌
const exampleToken =
  "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IuW4kOWPt-eUqOaItyIsImlhdCI6MTcwNjYwNDQwMCwiZXhwIjoxNzM4MTQwNDAwLCJhZG1pbiI6dHJ1ZX0.S-KBsqVNlWw-f9jw-rGO9VmQRXqFe_1g-QjsUw0TxbY";

// 插入示例
const insertExample = () => {
  form.token = exampleToken;
  parseToken();
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

.jwt-section {
  display: flex;
  flex-direction: column;
  gap: 24px;
}

.input-section {
  background: var(--el-bg-color-page);
  border-radius: 8px;
  padding: 20px;
}

.jwt-input {
  display: flex;
  gap: 8px;
}

.input-buttons {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.input-buttons .el-button {
  width: 100px;
}

.input-buttons .el-icon {
  margin-right: 4px;
}

.result-section {
  background: var(--el-bg-color-page);
  border-radius: 8px;
  padding: 20px;
  min-height: 200px;
}

.empty-state {
  height: 100%;
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

.jwt-part {
  margin-bottom: 24px;
  background: var(--el-bg-color);
  border-radius: 8px;
  padding: 16px;
}

.part-header {
  display: flex;
  align-items: center;
  gap: 12px;
  margin-bottom: 12px;
}

.part-header h3 {
  margin: 0;
  font-size: 16px;
  color: var(--el-text-color-primary);
}

.part-label {
  font-size: 12px;
  color: var(--el-text-color-secondary);
  background: var(--el-color-primary-light-9);
  padding: 2px 8px;
  border-radius: 4px;
}

.json-content {
  margin: 0;
  padding: 12px;
  background: var(--el-bg-color-page);
  border-radius: 4px;
  color: var(--el-text-color-regular);
  font-family: monospace;
  font-size: 14px;
  white-space: pre-wrap;
  word-break: break-all;
}

.signature-content {
  padding: 12px;
  background: var(--el-bg-color-page);
  border-radius: 4px;
  color: var(--el-text-color-regular);
  font-family: monospace;
  font-size: 14px;
  word-break: break-all;
}

.time-info {
  margin-top: 16px;
  padding: 12px;
  background: var(--el-bg-color-page);
  border-radius: 4px;
}

.time-item {
  display: flex;
  gap: 8px;
  margin-bottom: 8px;
}

.time-item:last-child {
  margin-bottom: 0;
}

.time-item .label {
  color: var(--el-text-color-secondary);
  font-size: 14px;
}

.time-item .value {
  color: var(--el-text-color-primary);
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
}
</style>
