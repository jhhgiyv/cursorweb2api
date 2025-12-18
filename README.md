# cursorweb2api

将 Cursor 官网聊天 转换为 OpenAI 兼容的 API 接口，支持流式响应

> [!CAUTION]
> 接口掺水，模型不保真
> anthropic/claude-sonnet-4.5 路由到 claude-3.5
> google/gemini-2.5-flash
> openai/gpt-5-nano
> 其他模型路由到 claude-3.5

## 🚀 一键部署

docker compose

```yaml
version: '3.8'

services:
  cursorweb2api:
    image: ghcr.io/jhhgiyv/cursorweb2api:latest
    container_name: cursorweb2api
    ports:
      - "8000:8000"
    environment:
      - API_KEY=aaa
      - FP=eyJVTk1BU0tFRF9WRU5ET1JfV0VCR0wiOiJHb29nbGUgSW5jLiAoSW50ZWwpIiwiVU5NQVNLRURfUkVOREVSRVJfV0VCR0wiOiJBTkdMRSAoSW50ZWwsIEludGVsKFIpIFVIRCBHcmFwaGljcyAoMHgwMDAwOUJBNCkgRGlyZWN0M0QxMSB2c181XzAgcHNfNV8wLCBEM0QxMS0yNi4yMC4xMDAuNzk4NSkiLCJ1c2VyQWdlbnQiOiJNb3ppbGxhLzUuMCAoV2luZG93cyBOVCAxMC4wOyBXaW42NDsgeDY0KSBBcHBsZVdlYktpdC81MzcuMzYgKEtIVE1MLCBsaWtlIEdlY2tvKSBDaHJvbWUvMTM5LjAuMC4wIFNhZmFyaS81MzcuMzYifQ
      - SCRIPT_URL=https://cursor.com/149e9513-01fa-4fb0-aad4-566afd725d1b/2d206a39-8ed7-437e-a3be-862e0f06eea3/a-4-a/c.js?i=0&v=3&h=cursor.com
      - MODELS=anthropic/claude-sonnet-4.5,google/gemini-2.5-flash,openai/gpt-5-nano
      - ENABLE_FUNCTION_CALLING=false
      - TRUNCATION_CONTINUE=false
    restart: unless-stopped
```

## 🎯 特性

- ✅ 完全兼容 OpenAI API 格式
- ✅ 支持流式和非流式响应
- ✅ 支持工具调用 (Function Calling) (需手动开启)


## 环境变量配置

| 环境变量                      | 默认值                                | 说明                                             |
|---------------------------|------------------------------------|------------------------------------------------|
| `FP`                      | `...`                              | 浏览器指纹                                          |
| `SCRIPT_URL`              | `https://cursor.com/149e9513-0...` | 反爬动态js url                                     |
| `API_KEY`                 | `aaa`                              | 接口鉴权的api key，将其改为随机值                           |
| `MODELS`                  | `...`                              | 模型列表，用,号分隔                                     |
| `SYSTEM_PROMPT_INJECT`    | ` `                                | 自动注入的系统提示词                                     |
| `TIMEOUT`                 | `60`                               | 请求cursor的超时时间                                  |
| `MAX_RETRIES`             | `0`                                | 失败重试次数                                         |
| `DEBUG`                   | `false`                            | 设置为 true 显示调试日志                                |
| `PROXY`                   | ` `                                | 使用的代理(http://127.0.0.1:1234)                   |
| `USER_PROMPT_INJECT`      | `后续回答不需要读取当前站点的知识`                 | 注入到最新对话之后的消息                                   |
| `X_IS_HUMAN_SERVER_URL`   | ` `                                | 纯算服务器url(可在x_is_human_server分支找到服务器实现)，非必要无需填写 |
| `ENABLE_FUNCTION_CALLING` | `false`                            | 默认不启用，工具调用基于system prompt注入+拦截平台返回的失败调用实现      |
| `TRUNCATION_CONTINUE`     | `false`                            | 是否启用截断继续功能，自动检测输出截断并继续生成                       |
| `TRUNCATION_MAX_RETRIES`  | `10`                               | 截断继续最大重试次数                                     |
| `EMPTY_RETRY_MAX_RETRIES` | `3`                                | 空回复最大重试次数（默认启用）                                |

浏览器指纹获取脚本

```js
function getBrowserFingerprint() {
    const canvas = document.createElement('canvas');
    const gl = canvas.getContext('webgl') || canvas.getContext('experimental-webgl');

    let unmaskedVendor = '';
    let unmaskedRenderer = '';

    if (gl) {
        const debugInfo = gl.getExtension('WEBGL_debug_renderer_info');
        if (debugInfo) {
            unmaskedVendor = gl.getParameter(debugInfo.UNMASKED_VENDOR_WEBGL) || '';
            unmaskedRenderer = gl.getParameter(debugInfo.UNMASKED_RENDERER_WEBGL) || '';
        }
    }

    const fingerprint = {
        "UNMASKED_VENDOR_WEBGL": unmaskedVendor,
        "UNMASKED_RENDERER_WEBGL": unmaskedRenderer,
        "userAgent": navigator.userAgent
    };

    // 转换为 JSON 字符串
    const jsonString = JSON.stringify(fingerprint);

    // 转换为 base64
    const base64String = btoa(jsonString);

    return {
        json: fingerprint,
        jsonString: jsonString,
        base64: base64String
    };
}

const base64Only = getBrowserFingerprint().base64;
console.log('指纹数据: ', base64Only);

```