# Agent Console · 纯前端版

一个完全在浏览器中运行的 LLM 对话工具台。支持 DeepSeek、Kimi、混元-lite 及任意 OpenAI 兼容接口，API Key 与聊天记录只保存在本地 localStorage，无后端、无追踪、源码全开源。

![License](https://img.shields.io/badge/license-MIT-blue.svg)

## 在线访问

👉 **[https://你的用户名.github.io/agent-console-frontend](https://你的用户名.github.io/agent-console-frontend)**

> 部署后请把 `你的用户名` 替换为实际 GitHub 用户名。

## 特性

- **纯前端**：单个 `index.html`，下载后双击即可打开，也可部署到任意静态托管。
- **多 Provider 切换**：内置 DeepSeek、Kimi（Moonshot）、混元-lite，支持自定义 OpenAI 兼容接口。
- **模型切换**：每个 Provider 可配置多个模型，顶栏下拉即切。
- **API Key 本地保存**：Key 写入浏览器 localStorage，不会上传到任何服务器。
- **对话历史本地存储**：自动保存会话列表和聊天记录。
- **流式输出**：默认使用 SSE 流式响应，实时显示 AI 回复。
- **代码查看面板**：点击代码块可放大查看、一键复制，支持语法高亮。
- **参数配置面板**：系统提示词、Temperature、Max Tokens、Top P、Base URL 覆盖。
- **插件扩展**：预留 `AgentConsole.registerPlugin()` 接口，可加载第三方 JS 插件。
- **无设备白名单**：此版本已完全移除访问码和设备白名单功能。

## 快速开始

### 方式一：直接打开

1. 下载本仓库源码。
2. 双击 `index.html` 用浏览器打开。
3. 点击左下角 **设置**，选择 Provider 并填入 API Key。
4. 顶栏选择模型，输入消息按 Enter 发送。

### 方式二：部署到 GitHub Pages

1. Fork 本仓库。
2. 进入仓库 **Settings → Pages**。
3. Source 选择 **GitHub Actions**。
4. 推送任意更改后，GitHub Actions 会自动部署到 `https://你的用户名.github.io/agent-console-frontend`。

## 使用说明

### 切换 Provider 和模型

- 顶栏左侧下拉选择 Provider（DeepSeek / Kimi / 混元-lite / 自定义）。
- 顶栏右侧下拉选择具体模型。

### 配置 API Key

1. 点击左下角 **设置**。
2. 选择当前 Provider。
3. 在 **API Key** 输入框填入 Key，点击 **保存**。
4. Key 仅保存在当前浏览器本地。

### 自定义 Provider

选择「自定义 (OpenAI 兼容)」后，在设置面板填写：

- **Base URL**：任意 OpenAI 兼容接口地址，例如 `https://your-api.com/v1`。
- **模型列表**：每行一个模型名。
- **API Key**：对应接口的 Key。

### 修改系统提示词

在设置面板顶部的「系统提示词」中修改，保存后对新会话生效。默认保留了原脚本中的毒舌犀利设定，可自由改成任何风格。

### 参数说明

| 参数 | 说明 | 范围 |
|------|------|------|
| Temperature | 随机性，越高越有创意 | 0 ~ 2 |
| Max Tokens | 单次回复最大 token 数 | 256 ~ 8192 |
| Top P | 核采样概率阈值 | 0 ~ 1 |

### 关于 CORS

浏览器直接请求 DeepSeek / Kimi / 混元等官方接口时，部分网络环境可能遇到 **CORS 跨域拦截**。解决方法：

1. 在设置中把 **Base URL** 改成你自己的转发代理（如 Cloudflare Worker、Nginx 反代）。
2. 或使用支持 CORS 的第三方 API 网关。
3. 本地测试时可用浏览器插件临时关闭 CORS（仅开发用途）。

## 插件开发

Agent Console 预留了简单的插件接口，可通过 URL 参数加载外部 JS 插件：

```
https://你的用户名.github.io/agent-console-frontend?plugin=https://example.com/my-plugin.js
```

插件示例 `my-plugin.js`：

```javascript
AgentConsole.registerPlugin({
  name: 'my-plugin',
  onRegister({ config, sessions, sendMessage }) {
    console.log('插件已加载', config);
  },
  onBeforeSend(payload, config) {
    // 在发送前修改请求体
    payload.temperature = 0.5;
  }
});
```

可用扩展点：

- `onRegister(context)`：插件注册时调用。
- `onBeforeSend(payload, config)`：发送请求前修改 payload。

未来会扩展更多扩展点（自定义渲染、自定义 Provider、工具调用等），欢迎 PR。

## 隐私与安全

- **无后端**：所有请求直接由浏览器发向你配置的 Provider。
- **本地存储**：API Key、聊天记录、配置均保存在浏览器 `localStorage`。
- **不上传**：没有任何数据会发送到本工具台的作者服务器。
- **谨慎共享**：不要把包含 Key 的 localStorage 或截图发送给他人。
- **一键清空**：设置面板底部提供「清空历史」和「清空所有数据」。

## 目录结构

```
agent-console-frontend/
├── index.html          # 完整单页应用（HTML + CSS + JS）
├── README.md           # 本文档
├── LICENSE             # MIT 协议
└── .github/workflows/
    └── pages.yml       # GitHub Pages 自动部署
```

## 贡献

欢迎 Issue 和 Pull Request。

## 开源协议

[MIT](./LICENSE)
