# hello-world-
Hello-world浏览器扩展
# Hello World 浏览器扩展程序完整教程

## 目录
1. [基础概念](#基础概念)
2. [项目结构](#项目结构)
3. [文件详解](#文件详解)
4. [安装步骤](#安装步骤)
5. [功能拓展](#功能拓展)
6. [调试技巧](#调试技巧)

---

## 基础概念

### 什么是浏览器扩展？
浏览器扩展是为浏览器添加新功能的小程序。可以：
- 修改网页内容
- 添加新工具栏按钮
- 管理标签页
- 拦截网络请求
- 存储数据

### Manifest 文件
`manifest.json` 是扩展的配置文件，告诉浏览器：
- 扩展的名称和版本
- 需要什么权限
- 包含哪些文件

### Manifest V3 vs V2
- **Manifest V3**（新标准）：更安全，Chrome推荐使用
- **Manifest V2**（旧标准）：已停用

---

## 项目结构

```
hello-world-extension/
│
├── manifest.json          # 配置文件（必需）
├── popup.html            # 弹窗界面
├── popup.js              # 弹窗逻辑
├── content.js            # 内容脚本（可选）
├── background.js         # 后台脚本（可选）
└── images/               # 图标文件夹
    ├── icon-16.png
    ├── icon-48.png
    └── icon-128.png
```

---

## 文件详解

### 1. manifest.json - 配置文件

**作用：** 定义扩展的基本信息和权限

```json
{
  "manifest_version": 3,              // 使用版本3
  "name": "Hello World Extension",    // 扩展名称
  "version": "1.0",                   // 版本号
  "description": "第一个扩展程序",     // 描述
  
  "permissions": [                    // 权限申请
    "activeTab",                      // 访问当前标签页
    "scripting",                       // 注入脚本
    "storage"                         // 使用存储空间
  ],
  
  "action": {                         // 点击图标的行为
    "default_popup": "popup.html",    // 打开的弹窗
    "default_title": "Hello World"    // 鼠标悬停提示
  },
  
  "icons": {                          // 扩展图标
    "16": "images/icon-16.png",
    "48": "images/icon-48.png",
    "128": "images/icon-128.png"
  },
  
  "background": {                     // 后台脚本（可选）
    "service_worker": "background.js"
  }
}
```

**关键参数说明：**
| 参数 | 说明 | 必需 |
|-----|------|------|
| manifest_version | 使用版本3 | ✓ |
| name | 扩展名称 | ✓ |
| version | 版本号 | ✓ |
| permissions | 权限列表 | ✗ |
| action | 图标点击行为 | ✗ |
| icons | 扩展图标 | ✗ |

### 2. popup.html - 用户界面

**作用：** 定义点击扩展图标时显示的界面

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>Hello World</title>
  <link rel="stylesheet" href="popup.css">
</head>
<body>
  <div class="container">
    <h1>👋 Hello World!</h1>
    <p>欢迎使用我的第一个扩展程序</p>
    
    <!-- 输入框 -->
    <input type="text" id="inputText" placeholder="输入文本">
    
    <!-- 按钮 -->
    <button id="saveButton">保存</button>
    <button id="clearButton">清空</button>
    
    <!-- 显示结果 -->
    <div id="result"></div>
  </div>
  
  <!-- 引入脚本 -->
  <script src="popup.js"></script>
</body>
</html>
```

**HTML 结构：**
- 使用标准 HTML5
- 必须引入 JavaScript 文件
- 不能使用内联脚本 `<script>console.log()</script>`

### 3. popup.js - 交互逻辑

**作用：** 处理弹窗中的交互和数据存储

```javascript
// ===== 页面加载完成 =====
document.addEventListener('DOMContentLoaded', function() {
  console.log('Popup 页面已加载');
  
  // 获取保存的数据并显示
  loadData();
  
  // 绑定事件监听
  bindEvents();
});

// ===== 绑定事件 =====
function bindEvents() {
  document.getElementById('saveButton').addEventListener('click', saveData);
  document.getElementById('clearButton').addEventListener('click', clearData);
}

// ===== 保存数据 =====
function saveData() {
  const inputText = document.getElementById('inputText').value;
  
  // 验证输入
  if (!inputText.trim()) {
    alert('请输入内容');
    return;
  }
  
  // 保存到 Chrome 存储
  chrome.storage.local.set({
    savedText: inputText,
    timestamp: new Date().toLocaleTimeString()
  }, function() {
    console.log('数据已保存');
    showResult('✓ 数据已保存！');
  });
}

// ===== 加载数据 =====
function loadData() {
  chrome.storage.local.get(['savedText'], function(result) {
    if (result.savedText) {
      document.getElementById('inputText').value = result.savedText;
    }
  });
}

// ===== 清空数据 =====
function clearData() {
  document.getElementById('inputText').value = '';
  
  chrome.storage.local.remove('savedText', function() {
    console.log('数据已清空');
    showResult('✓ 数据已清空！');
  });
}

// ===== 显示结果信息 =====
function showResult(message) {
  const resultDiv = document.getElementById('result');
  resultDiv.textContent = message;
  resultDiv.style.display = 'block';
  
  // 3秒后隐藏
  setTimeout(function() {
    resultDiv.style.display = 'none';
  }, 3000);
}
```

**JavaScript 关键概念：**

| 概念 | ���明 | 示例 |
|-----|------|------|
| `DOMContentLoaded` | 页面加载完成事件 | 初始化界面 |
| `chrome.storage.local` | 本地存储 | 保存用户数据 |
| `chrome.runtime` | 与后台脚本通信 | 发送消息 |
| `chrome.tabs` | 管理标签页 | 执行脚本 |

### 4. content.js - 网页内容脚本

**作用：** 在网页中注入脚本，操作网页内容

```javascript
// ===== 监听来自弹窗的消息 =====
chrome.runtime.onMessage.addListener(function(request, sender, sendResponse) {
  console.log('收到消息:', request.message);
  
  if (request.message === 'getPageInfo') {
    // 获取网页信息
    const pageInfo = {
      title: document.title,
      url: window.location.href,
      time: new Date().toLocaleTimeString()
    };
    
    sendResponse(pageInfo);
  }
});

// ===== 注入样式 =====
function injectStyles() {
  const style = document.createElement('style');
  style.textContent = `
    .hello-world-banner {
      position: fixed;
      top: 10px;
      right: 10px;
      background: #667eea;
      color: white;
      padding: 10px 15px;
      border-radius: 4px;
      z-index: 9999;
    }
  `;
  document.head.appendChild(style);
}

// ===== 页面加载完成 =====
if (document.readyState === 'loading') {
  document.addEventListener('DOMContentLoaded', injectStyles);
} else {
  injectStyles();
}

console.log('Content script 已注入页面');
```

### 5. background.js - 后台脚本

**作用：** 处理后台任务，不依赖于弹窗

```javascript
// ===== 扩展安装时 =====
chrome.runtime.onInstalled.addListener(function() {
  console.log('扩展已安装');
  
  // 打开欢迎页面
  chrome.tabs.create({
    url: 'welcome.html'
  });
});

// ===== 监听来自 content.js 的消息 =====
chrome.runtime.onMessage.addListener(function(request, sender, sendResponse) {
  if (request.type === 'logData') {
    console.log('收到日志:', request.data);
    sendResponse({ success: true });
  }
});

// ===== 定时任务 =====
chrome.alarms.create('dailyTask', { periodInMinutes: 24 * 60 });

chrome.alarms.onAlarm.addListener(function(alarm) {
  if (alarm.name === 'dailyTask') {
    console.log('执行每日任务');
  }
});
```

---

## 安装步骤

### Chrome 浏览器安装

**步骤 1：** 打开开发者模式
1. 在地址栏输入 `chrome://extensions/`
2. 点击右上角 **开发者模式** 开关

**步骤 2：** 加载扩展
1. 点击 **加载未打包的扩展程序**
2. 选择包含 `manifest.json` 的文件夹
3. 扩展会显示在列表中

**步骤 3：** 测试
1. 点击工具栏中的扩展图标
2. 弹窗应该会打开
3. 查看 **开发者工具** 查看日志

### Firefox 浏览器安装

**步骤 1：** 打开调试页面
1. 在地址栏输入 `about:debugging#/runtime/this-firefox`

**步骤 2：** 加载扩展
1. 点击 **加载临时附加组件**
2. 选择 `manifest.json` 文件

**步骤 3：** 测试
1. 扩展会显示在工具栏
2. 点击查看弹窗

---

## 功能拓展

### 1. 添加右键菜单

```javascript
// 在 manifest.json 中添加权限
"permissions": ["contextMenus"]

// 在 background.js 中创建菜单
chrome.contextMenus.create({
  id: "hello-world",
  title: "Hello World 菜单",
  contexts: ["selection"]
});

// 监听菜单点击
chrome.contextMenus.onClicked.addListener(function(info, tab) {
  if (info.menuItemId === "hello-world") {
    alert('您选择的文本: ' + info.selectionText);
  }
});
```

### 2. 修改网页内容

```javascript
// 在 content.js 中修改网页
// 获取所有段落
const paragraphs = document.querySelectorAll('p');

// 添加边框
paragraphs.forEach(function(p) {
  p.style.border = '2px solid red';
  p.style.padding = '10px';
});
```

### 3. 与后台脚本通信

```javascript
// 在 popup.js 中发送消息
chrome.runtime.sendMessage(
  { message: 'getData' },
  function(response) {
    console.log('收到响应:', response);
  }
);

// 在 background.js 中接收消息
chrome.runtime.onMessage.addListener(function(request, sender, sendResponse) {
  if (request.message === 'getData') {
    sendResponse({ data: 'Hello World!' });
  }
});
```

### 4. 存储用户设置

```javascript
// 保存设置
chrome.storage.sync.set({
  theme: 'dark',
  notifications: true
}, function() {
  console.log('设置已保存');
});

// 读取设置
chrome.storage.sync.get(['theme', 'notifications'], function(result) {
  console.log('主题:', result.theme);
  console.log('通知:', result.notifications);
});
```

---

## 调试技巧

### 查看控制台输出

**步骤：**
1. 打开 `chrome://extensions/`
2. 找到您的扩展
3. 点击 **Service Worker** 查看后台日志
4. 点击 **检查窗口** 查看弹窗日志

### 常见问题

| 问题 | 原因 | 解决方案 |
|-----|------|--------|
| 图标不显示 | 图片路径错误 | 检查 icons 路径 |
| 弹窗为空白 | HTML 加载失败 | 检查 popup.html 路径 |
| 权限被拒绝 | 未申请权限 | 在 manifest.json 添加权限 |
| 脚本无法运行 | manifest 版本错误 | 使用 manifest_version 3 |
| 数据未保存 | 存储权限不足 | 添加 storage 权限 |

### 测试清单

- [ ] 扩展可以安装
- [ ] 点击图标弹窗打开
- [ ] 输入框可以保存数据
- [ ] 清空按钮可以删除数据
- [ ] 刷新页面数据仍然存在
- [ ] 控制台没有错误信息
- [ ] 图标正常显示

---

## 总结

您已经学会了：
✓ 创建浏览器扩展的基础结构
✓ 理解 Manifest V3 配置
✓ 建立用户界面和交互
✓ 存储和管理数据
✓ 调试和测试扩展

**下一步：**
- 添加更多功能（右键菜单、快捷键等）
- 美化界面和用户体验
- 发布到 Chrome Web Store
- 学习高级 API（标签页管理、网络请求等）

祝您扩展开发愉快！🚀
