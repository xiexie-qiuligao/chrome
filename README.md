# Chrome DevTools MCP Continuous

这是一个基于 [ChromeDevTools/chrome-devtools-mcp](https://github.com/ChromeDevTools/chrome-devtools-mcp) 的增强版 fork，重点优化了 **长生命周期浏览器会话** 场景，适合：

- AI agent 接管已经打开、已经登录的 Chrome
- 多标签页持续操作
- 减少每次工具调用时的全量页面/target 重扫
- 让浏览器状态更接近“持续会话”，而不是“每次命令都重新认识浏览器”

这个版本不是官方包，仓库地址是：

- [xiexie-qiuligao/chrome](https://github.com/xiexie-qiuligao/chrome)

## 这个 fork 改了什么

相比上游版，这个 fork 重点做了持续会话相关增强：

- 把“每次工具调用都刷新整套浏览器状态”的热路径改成了 **按需刷新**
- 引入 **增量 page/target registry**
- 已知 tab 的变化优先走 **增量更新**
- DevTools 状态改成 **脏页更新**
- `newPage()` / `closePage()` 不再强制触发整套页面世界重建
- 为 page 增加更稳定的 `targetId` 身份跟踪

一句话总结：

**这个版本更适合 agent 长时间接管一个真实 Chrome，会话更连续，多 tab 更稳定。**

## 适合什么场景

推荐你在下面这些场景使用这个版本：

- 需要接管你已经打开的 Chrome
- 页面已经登录，不想每次重新登录
- 需要同时盯很多个标签页
- 需要 agent 连续点、填、抓、截图、调试，而不是偶尔操作一下

如果你只是要一个标准官方 DevTools MCP，上游版也可以。  
如果你更在意 **持续会话、多 tab、长时间 agent 操作**，这个 fork 更合适。

## 环境要求

- Node.js `20.19+`
- 更推荐 Node.js `22`
- Google Chrome 最新稳定版或兼容版本
- npm

## npm 包名

这个 fork 预留的 npm 包名是：

```bash
chrome-devtools-mcp-continuous
```

如果你还没有发布到 npm，可以先按下面的“源码构建方式”使用。  
发布后，队友就可以直接通过 `npx chrome-devtools-mcp-continuous` 使用。

## 从源码构建

### 1. 克隆仓库

```bash
git clone https://github.com/xiexie-qiuligao/chrome.git
cd chrome
```

### 2. 安装依赖

```bash
npm install --ignore-scripts
```

### 3. 构建

推荐用 Node 22 构建：

```bash
npx -y node@22 node_modules/typescript/bin/tsc
npx -y node@22 --experimental-strip-types scripts/post-build.ts
```

构建完成后，MCP 启动入口是：

```text
build/src/bin/chrome-devtools-mcp.js
```

## 最推荐的使用方式：连接已经打开的 Chrome

如果你希望保留：

- 登录态
- 当前 tab 状态
- 多标签页上下文
- 持续会话体验

最推荐的方式是：**先自己启动 Chrome，再让这个 MCP 连进去。**

### Windows 启动 Chrome

```powershell
& "C:\Program Files\Google\Chrome\Application\chrome.exe" --remote-debugging-port=9222 --user-data-dir=D:\chrome-mcp-profile
```

说明：

- `--remote-debugging-port=9222` 用来开放调试端口
- `--user-data-dir` 建议单独给一个目录，不要直接用你平时的默认日常浏览目录

## MCP 配置示例

### 方式 1：本地源码构建版

适合你自己和队友先用 GitHub 仓库 build。

#### Codex / 通用 TOML 风格配置

```toml
[mcp_servers.chrome-devtools-continuous]
command = "cmd"
args = [
  "/c",
  "npx",
  "-y",
  "node@22",
  "C:\\path\\to\\chrome\\build\\src\\bin\\chrome-devtools-mcp.js",
  "--browser-url=http://127.0.0.1:9222",
  "--no-usage-statistics"
]
env = { SystemRoot = "C:\\Windows", PROGRAMFILES = "C:\\Program Files" }
startup_timeout_ms = 30000
```

把 `C:\\path\\to\\chrome\\...` 替换成你本地仓库实际路径。

### 方式 2：npm 发布后直接使用

发布到 npm 后，可以直接这样配：

```json
{
  "mcpServers": {
    "chrome-devtools-continuous": {
      "command": "npx",
      "args": [
        "-y",
        "chrome-devtools-mcp-continuous",
        "--browser-url=http://127.0.0.1:9222",
        "--no-usage-statistics"
      ]
    }
  }
}
```

## 队友怎么用

### 方案 A：直接用 GitHub 源码

队友执行：

```bash
git clone https://github.com/xiexie-qiuligao/chrome.git
cd chrome
npm install --ignore-scripts
npx -y node@22 node_modules/typescript/bin/tsc
npx -y node@22 --experimental-strip-types scripts/post-build.ts
```

然后按上面的 MCP 配置，把入口指向：

```text
build/src/bin/chrome-devtools-mcp.js
```

### 方案 B：用 npm 包

你把包发布到 npm 后，队友就不需要 build 了，直接：

```bash
npx -y chrome-devtools-mcp-continuous --browser-url=http://127.0.0.1:9222
```

## 如何发布到 npm

先确保你已经：

- 拥有 npm 账号
- `npm login`
- 这个包名没有被占用

### 1. 先本地打包检查

```bash
npm pack
```

### 2. 正式发布

```bash
npm publish
```

如果以后你想限制成公开包，也可以补：

```bash
npm publish --access public
```

## 本仓库里和发布有关的关键文件

- [package.json](./package.json)
- [server.json](./server.json)
- [.mcp.json](./.mcp.json)
- [gemini-extension.json](./gemini-extension.json)

## 上游项目与文档

这个仓库仍然基于上游 Chrome DevTools MCP，以下文档依然有参考价值：

- [工具文档](./docs/tool-reference.md)
- [变更记录](./CHANGELOG.md)
- [故障排查](./docs/troubleshooting.md)
- [设计原则](./docs/design-principles.md)

## 许可协议

沿用上游协议：

- `Apache-2.0`

## 说明

这个 fork 的定位不是替代上游官方项目，而是把它改得更适合：

- agent 持续接管浏览器
- 多标签页长时间运行
- 已登录会话复用

如果你关注的是“持续会话”，这个版本会比上游版更顺手。
