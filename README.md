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

Agent Console 提供轻量插件接口，插件就是一个带 `name` 的普通对象，通过 `AgentConsole.registerPlugin(plugin)` 注册。当前提供两个扩展点：

- `onRegister({ config, sessions, sendMessage })`：插件注册时调用一次，可拿到当前配置、会话数组，以及 `sendMessage` 主动发消息函数。
- `onBeforeSend(payload, config)`：每次发请求**前**调用，可修改请求体（注入 system 提示、改 `temperature/top_p`、加 `tools` 等）。该回调在模型参数确定**之后**执行，因此你在插件里覆盖这些字段是生效的。

### 方式一：URL 参数加载（无需改源码，适合分享给别人）

把插件写成一个 `.js` 文件放到任意可访问的地址，然后访问时在网址后面加 `?plugin=该文件URL`：

```
https://你的域名或用户名.github.io/?plugin=https://example.com/plugins/my-plugin.js
```

页面会自动加载并执行该脚本。控制台会打印 `[AgentConsole] 插件已注册：xxx`。

### 方式二：写进 index.html（适合自己长期固定使用）

在 `index.html` 末尾、`init();` 调用之前，加一行注册代码即可：

```html
<script>
  AgentConsole.registerPlugin({
    name: '自动加人设',
    onBeforeSend(payload) {
      payload.messages.unshift({
        role: 'system',
        content: '你是一个严谨的中文助手，回答要分点、给例子。'
      });
    }
  });
  init();
</script>
```

### 完整插件示例（仓库内 `plugins/example-plugin.js`）

```javascript
(function () {
  const myPlugin = {
    name: '自动加人设',
    onRegister({ config }) {
      console.log('[插件] 已注册：', this.name, '当前模型：', config.providerId);
    },
    onBeforeSend(payload) {
      payload.messages.unshift({
        role: 'system',
        content: '请用通俗的比喻解释复杂概念。'
      });
    }
  };
  window.AgentConsole.registerPlugin(myPlugin);
})();
```

仓库已附带两个可直接参考的文件：

- `plugins/template.js`：空白模板，复制改名即可开发你自己的插件。
- `plugins/example-plugin.js`：真实可用示例，给每次请求注入"用通俗比喻解释"的约束。

> **安全提示**：`?plugin=` 本质是加载并执行**任意外部脚本**，等于把该脚本的完整权限交出去。请只加载你自己或信任来源的插件，不要点击他人发来的不明 `?plugin=` 链接。

### 在网站里一键管理插件（无需改代码）

打开左下角 **设置**，找到 **插件管理** 区域：

- **安装**：在输入框粘贴插件的 `.js` 地址，点「安装」即可加载（脚本需自行调用 `AgentConsole.registerPlugin`）。
- **启用 / 禁用**：每个插件前有复选框，勾选即启用、取消即禁用；禁用后该插件的 `onBeforeSend` 不再生效。状态保存在本地 `localStorage`，刷新后保留。
- **卸载**：点「卸载」从当前会话移除该插件。

你也可以在地址栏用 `?plugin=地址` 的方式让页面打开时自动加载插件，加载后同样会出现在管理列表里。

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
├── plugins/            # 插件示例与模板
│   ├── example-plugin.js  # 真实可用示例（注入系统提示）
│   ├── chat-background.js # 半透明图片聊天背景（文件/链接 + 透明度调节）
│   └── template.js        # 空白插件模板
└── .github/workflows/
    └── pages.yml       # GitHub Pages 自动部署
```

## 贡献

欢迎 Issue 和 Pull Request。

## 开源协议

[MIT](./LICENSE)
