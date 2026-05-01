# 删除 Status Bar 实现计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 彻底删除 status-bar HTML 元素及相关 JS 代码，避免 DOM 元素删除后 JS 报错。

**Architecture:** 移除 status-bar 组件（包括 HTML 标签、CSS 样式、JS 逻辑），不影响汇率换算核心功能。

**Tech Stack:** 纯前端 HTML/CSS/JS

---

## 文件变更概览

- 修改: `index.html` (同一文件，输出到原位置)

---

## 任务 1: 删除 HTML 中的 status-bar 标签

**文件:** `D:\ai-dev-git\exchange-rate\index.html`

- [ ] **Step 1: 定位并删除 status-bar HTML**

定位 lines 613-622 (实际行号):
```html
    <!-- Status Bar -->
    <div class="status-bar" id="statusBar" role="status" aria-live="polite" aria-atomic="true" style="display:none;">
      <div class="status-bar__info">
        <span id="statusText"></span>
      </div>
      <div class="status-bar__cache" id="cacheInfo">
        <span class="status-bar__cache-dot"></span>
        <span id="cacheText">frankfurter.app</span>
      </div>
    </div>
```

删除这段 HTML，保留其前后内容不变（前面是 source-section，后面是 cards-grid）。

---

## 任务 2: 删除 CSS 中的 .status-bar 相关样式

**文件:** `D:\ai-dev-git\exchange-rate\index.html`

- [ ] **Step 1: 定位并删除 status-bar CSS**

定位 lines 275-328 中的 `.status-bar` 样式块:
```css
    /* ─── Status Bar ─────────────────────────────────────────────── */
    .status-bar {
      display: flex;
      align-items: center;
      justify-content: space-between;
      flex-wrap: wrap;
      gap: 8px;
      padding: 10px 16px;
      background: var(--bg-card);
      border: 1px solid var(--border);
      border-radius: var(--radius-sm);
      font-size: 12px;
      animation: fadeDown 0.6s 0.15s ease both;
    }
    .status-bar__info {
      display: flex;
      align-items: center;
      gap: 8px;
      color: var(--text-mid);
    }
    .status-bar__info strong { color: var(--text-hi); font-weight: 600; }
    .status-bar__cache {
      display: flex;
      align-items: center;
      gap: 6px;
      color: var(--text-lo);
      font-size: 11px;
    }
    .status-bar__cache.is-cached { color: var(--accent); }
    .status-bar__cache-dot {
      width: 5px; height: 5px; border-radius: 50%; background: var(--text-lo);
    }
    .status-bar__cache.is-cached .status-bar__cache-dot { background: var(--accent); }
    .status-bar__msg {
      font-size: 12px;
      padding: 3px 10px;
      border-radius: 20px;
      font-weight: 500;
    }
    .status-bar__msg--warn {
      background: rgba(255, 200, 0, 0.1);
      border: 1px solid rgba(255, 200, 0, 0.25);
      color: #b37400;
    }
    .status-bar__msg--error {
      background: rgba(255, 59, 48, 0.1);
      border: 1px solid rgba(255, 59, 48, 0.25);
      color: var(--red);
    }
    .status-bar__msg--cached {
      background: var(--accent-dim);
      border: 1px solid var(--border);
      color: var(--accent);
    }
```

删除整个 `.status-bar` CSS 块（从 `/* ─── Status Bar ─── */` 到 `/* ─── Error Hint ─── */` 之前）。

---

## 任务 3: 删除 JS 中的元素引用和函数

**文件:** `D:\ai-dev-git\exchange-rate\index.html`

- [ ] **Step 1: 删除 Elements 部分中的 statusBar, statusText, cacheInfo, cacheText 引用**

定位 line 681-684，删除:
```javascript
    const statusBar        = document.getElementById('statusBar');
    const statusText        = document.getElementById('statusText');
    const cacheInfo         = document.getElementById('cacheInfo');
    const cacheText         = document.getElementById('cacheText');
```

- [ ] **Step 2: 删除 updateCacheUI 函数**

定位 lines 797-805，删除整个函数:
```javascript
    function updateCacheUI(fromCache) {
      if (fromCache) {
        cacheInfo.classList.add('is-cached');
        cacheText.textContent = '缓存数据';
      } else {
        cacheInfo.classList.remove('is-cached');
        cacheText.textContent = 'frankfurter.app';
      }
    }
```

- [ ] **Step 3: 删除 updateStatus 函数**

定位 lines 807-825，删除整个函数:
```javascript
    function updateStatus(msg, type) {
      // Remove old type classes
      statusBar.querySelectorAll('.status-bar__msg').forEach(el => el.remove());
      if (!msg) return;

      // Extract leading icon if present
      const iconMatch = msg.match(/^([^\w]+)\s*(.*)/);
      let icon = '';
      let text = msg;
      if (iconMatch && !/^[a-z]/i.test(iconMatch[1])) {
        icon = iconMatch[1];
        text = iconMatch[2];
      }

      const badge = document.createElement('span');
      badge.className = `status-bar__msg status-bar__msg--${type || 'info'}`;
      badge.textContent = `${icon} ${text}`.trim();
      statusText.parentElement.insertBefore(badge, statusText.nextSibling);
    }
```

- [ ] **Step 4: 清理所有 updateStatus 调用**

在 `handleInput` 函数 (line 927 开始) 中，删除/替换以下调用:

1. Line 943: `updateStatus('', '');` → 删除此行
2. Line 956: `updateStatus('正在获取汇率...', '');` → 删除此行
3. Line 964: `updateStatus(...)` → 删除此行
4. Line 966: `updateStatus('', '');` → 删除此行
5. Line 974: `updateStatus(...)` → 删除此行
6. Line 982: `updateStatus(...)` → 删除此行

- [ ] **Step 5: 清理 updateCacheUI 调用**

在 `getRates` 函数中 (line 765 开始):
1. Line 782: `updateCacheUI(true);` → 删除此行
2. Line 788: `updateCacheUI(false);` → 删除此行

- [ ] **Step 6: 删除 checkMarketStatus 函数并调用**

定位 lines 1016-1022，删除整个函数:
```javascript
    function checkMarketStatus() {
      const now = new Date();
      const day = now.getDay(); // 0=Sun, 6=Sat
      if (day === 0 || day === 6) {
        updateStatus('周末非交易日，数据可能延迟', 'warn');
      }
    }
```

并删除 line 1025 的初始化调用:
```javascript
    checkMarketStatus();
```

---

## 验证清单

- [ ] HTML 中无 `id="statusBar"`, `id="statusText"`, `id="cacheInfo"`, `id="cacheText"` 元素
- [ ] CSS 中无 `.status-bar` 相关样式
- [ ] JS 中无 `updateStatus`, `updateCacheUI`, `checkMarketStatus` 函数定义
- [ ] JS 中无 `statusBar`, `statusText`, `cacheInfo`, `cacheText` 变量引用
- [ ] 页面加载无 JS 报错
- [ ] 汇率换算功能正常工作
- [ ] 文件末尾无多余内容（检查 `</html>` 之后）
