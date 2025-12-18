# 实战案例：Chrome 扩展

> 难度：⭐⭐ 中等 | 预计时间：2-3 小时 | 技术栈：JavaScript + Chrome API

## 🎯 项目目标

构建一个实用的 Chrome 扩展 - **网页笔记助手**：
- 选中文本快速保存
- 自动记录来源 URL
- 本地存储笔记
- 导出功能

## 📋 开始前的准备

### 环境要求
- Chrome 浏览器
- 代码编辑器

### 第一步：需求澄清

复制以下提示词给 AI：

```
我想用 Vibe Coding 的方式开发一个 Chrome 扩展 - 网页笔记助手。

技术要求：
- Manifest V3
- 纯 JavaScript（不用框架）
- Chrome Storage API

功能需求：
1. 右键菜单：选中文本后右键保存
2. 弹出窗口：显示所有笔记
3. 自动记录：来源 URL、保存时间
4. 导出功能：导出为 JSON/Markdown

请帮我：
1. 确认技术方案
2. 生成项目结构
3. 一步步指导我完成开发

要求：每完成一步问我是否成功，再继续下一步。
```

## 🏗️ 项目结构

```
web-notes/
├── manifest.json           # 扩展配置
├── background.js           # 后台脚本
├── popup/
│   ├── popup.html         # 弹出窗口
│   ├── popup.css
│   └── popup.js
├── icons/
│   ├── icon16.png
│   ├── icon48.png
│   └── icon128.png
└── README.md
```

## 🔧 核心代码

### 扩展配置 (manifest.json)

```json
{
  "manifest_version": 3,
  "name": "网页笔记助手",
  "version": "1.0.0",
  "description": "快速保存网页内容到本地笔记",
  "permissions": [
    "storage",
    "contextMenus",
    "activeTab"
  ],
  "background": {
    "service_worker": "background.js"
  },
  "action": {
    "default_popup": "popup/popup.html",
    "default_icon": {
      "16": "icons/icon16.png",
      "48": "icons/icon48.png",
      "128": "icons/icon128.png"
    }
  },
  "icons": {
    "16": "icons/icon16.png",
    "48": "icons/icon48.png",
    "128": "icons/icon128.png"
  }
}
```

### 后台脚本 (background.js)

```javascript
// 创建右键菜单
chrome.runtime.onInstalled.addListener(() => {
  chrome.contextMenus.create({
    id: 'saveNote',
    title: '保存到笔记',
    contexts: ['selection']
  });
});

// 处理右键菜单点击
chrome.contextMenus.onClicked.addListener(async (info, tab) => {
  if (info.menuItemId === 'saveNote' && info.selectionText) {
    const note = {
      id: Date.now().toString(),
      text: info.selectionText,
      url: tab.url,
      title: tab.title,
      createdAt: new Date().toISOString()
    };
    
    // 获取现有笔记
    const { notes = [] } = await chrome.storage.local.get('notes');
    
    // 添加新笔记
    notes.unshift(note);
    
    // 保存
    await chrome.storage.local.set({ notes });
    
    // 显示通知（可选）
    console.log('笔记已保存:', note.text.slice(0, 50));
  }
});
```

### 弹出窗口 HTML (popup/popup.html)

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <link rel="stylesheet" href="popup.css">
</head>
<body>
  <div class="container">
    <header>
      <h1>📝 我的笔记</h1>
      <div class="actions">
        <button id="exportJson">导出 JSON</button>
        <button id="exportMd">导出 MD</button>
        <button id="clearAll">清空</button>
      </div>
    </header>
    
    <div class="search">
      <input type="text" id="searchInput" placeholder="搜索笔记...">
    </div>
    
    <div id="notesList" class="notes-list">
      <!-- 笔记列表 -->
    </div>
    
    <footer>
      <span id="noteCount">0 条笔记</span>
    </footer>
  </div>
  
  <script src="popup.js"></script>
</body>
</html>
```

### 弹出窗口样式 (popup/popup.css)

```css
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  width: 400px;
  max-height: 500px;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
}

.container {
  padding: 16px;
}

header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 12px;
}

header h1 {
  font-size: 18px;
}

.actions button {
  padding: 4px 8px;
  margin-left: 4px;
  font-size: 12px;
  cursor: pointer;
  border: 1px solid #ddd;
  border-radius: 4px;
  background: #fff;
}

.actions button:hover {
  background: #f5f5f5;
}

.search input {
  width: 100%;
  padding: 8px 12px;
  border: 1px solid #ddd;
  border-radius: 4px;
  margin-bottom: 12px;
}

.notes-list {
  max-height: 350px;
  overflow-y: auto;
}

.note-item {
  padding: 12px;
  border: 1px solid #eee;
  border-radius: 8px;
  margin-bottom: 8px;
}

.note-text {
  font-size: 14px;
  line-height: 1.5;
  margin-bottom: 8px;
}

.note-meta {
  font-size: 12px;
  color: #666;
}

.note-meta a {
  color: #0066cc;
  text-decoration: none;
}

.note-delete {
  float: right;
  cursor: pointer;
  color: #999;
}

.note-delete:hover {
  color: #ff4444;
}

footer {
  margin-top: 12px;
  text-align: center;
  color: #666;
  font-size: 12px;
}

.empty {
  text-align: center;
  color: #999;
  padding: 40px;
}
```

### 弹出窗口逻辑 (popup/popup.js)

```javascript
let allNotes = [];

// 加载笔记
async function loadNotes() {
  const { notes = [] } = await chrome.storage.local.get('notes');
  allNotes = notes;
  renderNotes(notes);
}

// 渲染笔记列表
function renderNotes(notes) {
  const container = document.getElementById('notesList');
  const countEl = document.getElementById('noteCount');
  
  countEl.textContent = `${notes.length} 条笔记`;
  
  if (notes.length === 0) {
    container.innerHTML = '<div class="empty">暂无笔记<br>选中网页文字后右键保存</div>';
    return;
  }
  
  container.innerHTML = notes.map(note => `
    <div class="note-item" data-id="${note.id}">
      <span class="note-delete" onclick="deleteNote('${note.id}')">✕</span>
      <div class="note-text">${escapeHtml(note.text)}</div>
      <div class="note-meta">
        <a href="${note.url}" target="_blank">${note.title || '未知页面'}</a>
        <br>
        ${formatDate(note.createdAt)}
      </div>
    </div>
  `).join('');
}

// 删除笔记
async function deleteNote(id) {
  allNotes = allNotes.filter(n => n.id !== id);
  await chrome.storage.local.set({ notes: allNotes });
  renderNotes(allNotes);
}

// 搜索
document.getElementById('searchInput').addEventListener('input', (e) => {
  const query = e.target.value.toLowerCase();
  const filtered = allNotes.filter(note => 
    note.text.toLowerCase().includes(query) ||
    (note.title && note.title.toLowerCase().includes(query))
  );
  renderNotes(filtered);
});

// 导出 JSON
document.getElementById('exportJson').addEventListener('click', () => {
  const blob = new Blob([JSON.stringify(allNotes, null, 2)], { type: 'application/json' });
  downloadBlob(blob, 'notes.json');
});

// 导出 Markdown
document.getElementById('exportMd').addEventListener('click', () => {
  const md = allNotes.map(note => 
    `## ${formatDate(note.createdAt)}\n\n${note.text}\n\n> 来源: [${note.title}](${note.url})\n`
  ).join('\n---\n\n');
  const blob = new Blob([md], { type: 'text/markdown' });
  downloadBlob(blob, 'notes.md');
});

// 清空
document.getElementById('clearAll').addEventListener('click', async () => {
  if (confirm('确定清空所有笔记？')) {
    allNotes = [];
    await chrome.storage.local.set({ notes: [] });
    renderNotes([]);
  }
});

// 工具函数
function escapeHtml(text) {
  const div = document.createElement('div');
  div.textContent = text;
  return div.innerHTML;
}

function formatDate(isoString) {
  return new Date(isoString).toLocaleString('zh-CN');
}

function downloadBlob(blob, filename) {
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url;
  a.download = filename;
  a.click();
  URL.revokeObjectURL(url);
}

// 初始化
loadNotes();
```

## 🚀 安装步骤

1. 打开 Chrome，访问 `chrome://extensions/`
2. 开启「开发者模式」
3. 点击「加载已解压的扩展程序」
4. 选择项目文件夹

## ✅ 验收清单

- [ ] 扩展成功加载
- [ ] 右键菜单显示「保存到笔记」
- [ ] 选中文本可以保存
- [ ] 弹出窗口显示笔记列表
- [ ] 搜索功能正常
- [ ] 删除功能正常
- [ ] 导出 JSON 正常
- [ ] 导出 Markdown 正常

## 💡 进阶挑战

- 添加标签分类
- 支持云同步（Chrome Sync）
- 添加快捷键
- 支持图片保存
- 添加笔记编辑功能
