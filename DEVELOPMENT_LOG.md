# AudioVisual 开发日志

## 项目概述

- **原项目**: [RemotePinee/AudioVisual](https://github.com/RemotePinee/AudioVisual)
- **开发仓库**: [DYKJLL/AudioVisual-Optimized-v2](https://github.com/DYKJLL/AudioVisual-Optimized-v2)
- **备份仓库**: [DYKJLL/BFV2](https://github.com/DYKJLL/BFV2)
- **当前版本**: v1.2.1
- **开发日期**: 2026-04-11

---

## 版本历史

### v1.1.0 - 初始优化版本

**优化内容**:
1. **启动优化**: 后台异步预加载，不再阻塞主线程，窗口立即显示
2. **模块切换动画**: 添加 0.5s 淡入淡出过渡动画
3. **模块重置机制**: 切换模块时自动停止当前任务、清空浏览历史、返回首页
4. **视图缓存优化**: 缓存所有 9 个平台首页，切换响应更快速
5. **加载时间控制**: 加载超时机制确保 2 秒内完成
6. **启动默认模块**: 左侧保持美韩日剧选中状态，右侧显示腾讯视频首页

**修改的文件**:
- `main.js`: 预加载逻辑、IPC 处理器
- `preload-ui.js`: 添加 resetModule API
- `renderer.js`: 淡入淡出动画、启动模块
- `style.css`: 过渡动画样式

### v1.2.0 - 影巢加载优化

**问题**: 影巢模块加载时间 40-60 秒，或加载不出来

**优化内容**:
1. **预加载超时配置**: 影巢 20秒超时 + 3次重试；其他戏剧站点 15秒超时 + 2次重试
2. **加载超时处理**: 超时后自动发送 load-finished，避免无限等待
3. **浏览器优化**: 禁用 webSecurity 和 backgroundThrottling

**修改的文件**:
- `main.js`: 预加载重试机制
- `renderer.js`: 戏剧站点超时配置

### v1.2.1 - 戏剧网站体验优化（当前版本）

**问题**: 影巢加载时间 15-35 秒，点击剧集响应慢 1-2 秒

**优化内容**:
1. **广告拦截**: 阻止广告和追踪脚本加载，减少资源消耗
2. **加载遮罩**: 点击链接显示加载动画，提供视觉反馈
3. **图片懒加载**: 延迟加载图片，加速页面渲染
4. **链接预连接**: 鼠标悬停时预连接目标页面
5. **超时保护**: 10秒后自动隐藏加载遮罩

**修改的文件**:
- `preload-web.js`: DramaSiteOptimizer 模块

---

## 当前状态

### 功能模块

| 模块 | 状态 | 说明 |
|------|------|------|
| 启动加载 | 已优化 | 异步预加载，窗口立即显示 |
| 模块切换 | 已优化 | 淡入淡出动画，任务终止 |
| 国内解析平台 | 正常 | 腾讯、爱奇艺、优酷、B站、芒果TV |
| 美韩日剧模式 | 已优化 | 加载体验改善，但服务器延迟无法消除 |

### 已知限制

**影巢等戏剧网站加载慢的原因**:
1. 服务器在海外，物理延迟 100-300ms
2. DNS 解析、TCP 连接、TLS 握手需要额外时间
3. 网站资源较多，包含广告、追踪脚本
4. 无法通过代码优化完全消除物理延迟

**现实目标**:
- 加载时间: 从 40-60 秒优化到 5-15 秒
- 点击响应: 即时显示加载动画
- 超时保护: 最长等待 10 秒

---

## 代码结构

```
AudioVisual/
├── main.js                 # 主进程
│   ├── preloadAllSites()   # 后台预加载所有站点
│   ├── navigate IPC        # 导航处理
│   └── reset-module IPC    # 模块重置
├── assets/
│   ├── js/
│   │   ├── renderer.js     # 渲染进程主逻辑
│   │   ├── preload-ui.js   # UI 预加载脚本
│   │   └── preload-web.js  # 网页预加载脚本
│   └── css/
│       └── style.css       # 样式文件
├── index.html              # 主界面
└── package.json            # 项目配置
```

---

## 关键代码片段

### 1. 戏剧站点配置 (main.js)

```javascript
const dramaSites = [
  { url: 'https://monkey-flix.com/', name: '猴影工坊', timeout: 15000, retry: 2 },
  { url: 'https://www.movie1080.xyz/', name: '影巢movie', timeout: 20000, retry: 3 },
  { url: 'https://www.letu.me/', name: '茉小影', timeout: 15000, retry: 2 },
  { url: 'https://www.ncat21.com/', name: '网飞猫', timeout: 15000, retry: 2 }
];
```

### 2. 模块重置 IPC (main.js)

```javascript
ipcMain.on('reset-module', (event, url) => {
  if (view) {
    view.webContents.stop();
    view.webContents.setAudioMuted(true);
    if (view.webContents.clearHistory) {
      view.webContents.clearHistory();
    }
    mainWindow.removeBrowserView(view);
  }
  // 从缓存获取或创建新视图
  // ...
});
```

### 3. 戏剧网站优化器 (preload-web.js)

```javascript
const DramaSiteOptimizer = {
  isDramaSite() { /* ... */ },
  blockUnnecessaryResources() { /* ... */ },
  showLoadingOverlay() { /* ... */ },
  hideLoadingOverlay() { /* ... */ },
  prefetchLinks() { /* ... */ },
  init() { /* ... */ }
};
```

---

## 用户反馈记录

### 2026-04-11 会话记录

1. **初始问题**: 软件启动慢，模块切换卡顿，加载时间长
2. **第一轮优化**: v1.1.0 基础性能优化
3. **反馈**: 影巢加载 40-60 秒
4. **第二轮优化**: v1.2.0 预加载重试机制
5. **反馈**: 影巢加载 15-35 秒，有提升但仍有问题
6. **第三轮优化**: v1.2.1 广告拦截、加载动画
7. **最终决定**: 回退到 v1.2.1，后续可考虑其他方案

---

## 待优化项

### 短期（可选）
- [ ] 添加文字动效加载动画（已开发但回退）
- [ ] 在软件中添加"海外站点加载较慢"提示

### 长期（需要额外资源）
- [ ] 使用代理/CDN 加速（需要付费服务器）
- [ ] 寻找国内镜像站点

---

## 技术栈

- **框架**: Electron v33.4.11
- **前端**: HTML/CSS/JavaScript
- **打包**: electron-builder

---

## 构建命令

```bash
# 安装依赖
npm install

# 开发模式
npm start

# 打包 Windows 版本
npm run dist:win
```

---

## 下载链接

- **最新版本**: https://github.com/DYKJLL/AudioVisual-Optimized-v2/releases
- **备份版本**: https://github.com/DYKJLL/BFV2/releases

---

## 给下一个 AI 的建议

1. **不要过度优化戏剧网站**: 服务器物理延迟无法通过代码消除
2. **保持模块化**: 每次只修改一个模块，确认后再继续
3. **测试后确认**: 每次修改后需要用户测试确认
4. **版本控制**: 重大修改前先备份当前版本
5. **用户沟通**: 每次修改前确认修改清单，避免误解

---

## 仓库说明

| 仓库 | 用途 | 说明 |
|------|------|------|
| AudioVisual-Optimized-v2 | 开发仓库 | 日常开发使用 |
| BFV2 | 备份仓库 | 只读，仅在用户要求时更新 |
