# 视觉身份升级 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 落地视觉身份升级——F2 环点标图标套件、T2 侧栏标题、C2 工程蓝、Inter 自托管字体、头像精修，全部以站点级覆盖实现。

**Architecture:** 零接触并行会话文件（`assets/css/jekyll-theme-chirpy.scss`、`_tabs/friends.md`）。字体栈经主题钩子 `_sass/variables-hook.scss` 注入；`@font-face` 与全部视觉覆盖放进 `assets/css/identity.css`，经 `_data/origin/cors.yml` 的 `webfonts` 槽位加载（主题 `basic.yml` 同款机制，加载位在主题主样式之后）；模板层用 `_includes/favicons.html`、`_includes/sidebar.html` 覆盖；二进制资产（favicon 套件、Inter woff2、精修头像）由一次性 Node+CDP 无头 Chrome 脚本生成入库。

**Tech Stack:** Jekyll 4.3.3 + jekyll-theme-chirpy 7.0.1（gem）、Node 24（CDP 脚本）、无头 Chrome（资产渲染与核验）、Git Bash。

**设计依据：** `docs/superpowers/specs/2026-09-17-visual-identity-design.md`（已确认并提交 7cdf7ea）。

**环境事实（执行前已知）：**
- 构建命令（必须用绝对路径的 bundle，裸 `bundle` 会命中 WindowsApps 存根）：`bash /d/Ruby33-x64/bin/bundle exec jekyll build`，在 `D:\code\blog` 下执行。
- Chrome：`C:/Program Files/Google/Chrome/Application/chrome.exe`（`--headless=new` + `--remote-debugging-port`，独立 `--user-data-dir`，**绝不动用户正在开的 Chrome**）。
- python 预览服务：`/c/Python314/python -m http.server PORT`，后台跑、记 PID，**不要管道输出**。
- 临时工作目录：`C:/Users/hack9/AppData/Local/Temp/blog-shots/`（脚本与核验页面都放这里，不入库）。
- 当前工作区已有并行会话的未提交改动（`_tabs/friends.md`、`assets/css/jekyll-theme-chirpy.scss`）——**任何 commit 只按显式路径 add，绝不用 `git add -A`**。
- `_config.yml` 中 `pwa.enabled: true`、`assets.self_host.enabled` 为空（即 cors 型资源）。站点无 `_sass`、`assets/img` 目录（须新建）。

---

### Task 1: F2 环点标图标套件（生成 + 模板接入）

**Files:**
- Create: `assets/img/favicons/` 下 10 个文件（favicon.svg、favicon-16x16.png、favicon-32x32.png、favicon.ico、apple-touch-icon.png、android-chrome-192x192.png、android-chrome-512x512.png、mstile-150x150.png、site.webmanifest、browserconfig.xml）
- Create: `_includes/favicons.html`（站点覆盖主题同名文件）
- Scratch: `C:/Users/hack9/AppData/Local/Temp/blog-shots/gen-favicons.mjs`、`favicon-check.html`、`favicons/`（核验副本）

- [ ] **Step 1: 写生成脚本 `gen-favicons.mjs`**

写入 `C:/Users/hack9/AppData/Local/Temp/blog-shots/gen-favicons.mjs`：

```js
// 一次性脚本：无头 Chrome canvas 渲染 F2 环点标 → PNG/ICO（透明角保真）
import { setTimeout as sleep } from 'node:timers/promises';
import { spawn } from 'node:child_process';
import fs from 'node:fs';

const PORT = 9336;
const CHROME = 'C:/Program Files/Google/Chrome/Application/chrome.exe';
const OUT = 'D:/code/blog/assets/img/favicons';
fs.mkdirSync(OUT, { recursive: true });

const chrome = spawn(CHROME, [
  '--headless=new', '--disable-gpu', '--hide-scrollbars', '--no-first-run', '--no-default-browser-check',
  `--remote-debugging-port=${PORT}`,
  '--user-data-dir=C:/Users/hack9/AppData/Local/Temp/chrome-favicon-profile',
  'about:blank',
], { stdio: 'ignore' });

async function waitVersion() {
  for (let i = 0; i < 50; i++) {
    try {
      const r = await fetch(`http://127.0.0.1:${PORT}/json/version`);
      if (r.ok) return;
    } catch {}
    await sleep(300);
  }
  throw new Error('chrome not up');
}
await waitVersion();

const resp = await fetch(`http://127.0.0.1:${PORT}/json/new?about:blank`, { method: 'PUT' });
const target = await resp.json();
const ws = new WebSocket(target.webSocketDebuggerUrl);
let id = 0;
const pend = new Map();
const send = (method, params = {}) =>
  new Promise((res, rej) => {
    const i = ++id;
    pend.set(i, { res, rej });
    ws.send(JSON.stringify({ id: i, method, params }));
  });
await new Promise((r) => (ws.onopen = r));
ws.onmessage = (ev) => {
  const m = JSON.parse(ev.data);
  if (m.id && pend.has(m.id)) {
    const { res, rej } = pend.get(m.id);
    pend.delete(m.id);
    m.error ? rej(new Error(JSON.stringify(m.error))) : res(m.result);
  }
};

// 变体：small = 小尺寸光学校准（加粗）；normal = 标准；square = 满幅方底（Apple/Win 瓦片）
const jobs = [
  ['favicon-16x16.png', 'small', 16],
  ['favicon-32x32.png', 'small', 32],
  ['favicon-48x48.png', 'normal', 48],
  ['mstile-150x150.png', 'square', 150],
  ['apple-touch-icon.png', 'square', 180],
  ['android-chrome-192x192.png', 'normal', 192],
  ['android-chrome-512x512.png', 'normal', 512],
];

const renderExpr = `(async () => {
  const mk = (stroke, dot, rx) =>
    '<svg xmlns="http://www.w3.org/2000/svg" width="64" height="64" viewBox="0 0 64 64">' +
    '<rect width="64" height="64" rx="' + rx + '" fill="#15161a"/>' +
    '<path d="M44.26 21.72 A16 16 0 1 0 44.26 42.28" fill="none" stroke="#60A5FA" stroke-width="' + stroke + '" stroke-linecap="round"/>' +
    '<circle cx="48" cy="32" r="' + dot + '" fill="#60A5FA"/>' +
    '</svg>';
  const params = { small: [8.4, 5.8, 14], normal: [6.4, 4.6, 14], square: [6.4, 4.6, 0] };
  const jobs = ${JSON.stringify(jobs)};
  const out = {};
  for (const [name, variant, size] of jobs) {
    const img = new Image();
    img.src = 'data:image/svg+xml;base64,' + btoa(mk(...params[variant]));
    await img.decode();
    const c = document.createElement('canvas');
    c.width = size; c.height = size;
    c.getContext('2d').drawImage(img, 0, 0, size, size);
    out[name] = c.toDataURL('image/png').slice('data:image/png;base64,'.length);
  }
  return out;
})()`;

const r = await send('Runtime.evaluate', { expression: renderExpr, awaitPromise: true, returnByValue: true });
if (!r.result.value) throw new Error('render failed: ' + JSON.stringify(r));
const bufs = {};
for (const [name, b64] of Object.entries(r.result.value)) {
  bufs[name] = Buffer.from(b64, 'base64');
  if (name !== 'favicon-48x48.png') {
    fs.writeFileSync(`${OUT}/${name}`, bufs[name]);
    console.log('wrote', name, bufs[name].length, 'bytes');
  }
}

// ICO：16+32+48 PNG 内嵌（PNG-in-ICO，所有现代浏览器支持）
function buildIco(entries) {
  const header = Buffer.alloc(6);
  header.writeUInt16LE(0, 0);
  header.writeUInt16LE(1, 2);
  header.writeUInt16LE(entries.length, 4);
  let offset = 6 + entries.length * 16;
  const parts = [header];
  for (const { size, buf } of entries) {
    const e = Buffer.alloc(16);
    e.writeUInt8(size >= 256 ? 0 : size, 0);
    e.writeUInt8(size >= 256 ? 0 : size, 1);
    e.writeUInt8(0, 2);
    e.writeUInt8(0, 3);
    e.writeUInt16LE(1, 4);
    e.writeUInt16LE(32, 6);
    e.writeUInt32LE(buf.length, 8);
    e.writeUInt32LE(offset, 12);
    parts.push(e);
    offset += buf.length;
  }
  for (const { buf } of entries) parts.push(buf);
  return Buffer.concat(parts);
}

const ico = buildIco([
  { size: 16, buf: bufs['favicon-16x16.png'] },
  { size: 32, buf: bufs['favicon-32x32.png'] },
  { size: 48, buf: bufs['favicon-48x48.png'] },
]);
fs.writeFileSync(`${OUT}/favicon.ico`, ico);
console.log('wrote favicon.ico', ico.length, 'bytes');

await send('Browser.close').catch(() => {});
process.exit(0);
```

- [ ] **Step 2: 运行脚本并核对产出**

Run:
```bash
cd "/c/Users/hack9/AppData/Local/Temp/blog-shots" && node gen-favicons.mjs && ls -la "D:/code/blog/assets/img/favicons"
```
Expected: 7 个 PNG + favicon.ico，全部 >0 字节；favicon-16x16.png 通常 <1KB，android-chrome-512x512.png 数 KB 级。若报 `chrome not up`，重跑一次（端口占用时先等 2 秒）。

- [ ] **Step 3: 写 favicon.svg（矢量版，经 Write 工具）**

`D:\code\blog\assets\img\favicons\favicon.svg`：

```svg
<svg xmlns="http://www.w3.org/2000/svg" width="64" height="64" viewBox="0 0 64 64">
  <rect width="64" height="64" rx="14" fill="#15161a"/>
  <path d="M44.26 21.72 A16 16 0 1 0 44.26 42.28" fill="none" stroke="#60A5FA" stroke-width="7" stroke-linecap="round"/>
  <circle cx="48" cy="32" r="5" fill="#60A5FA"/>
</svg>
```

- [ ] **Step 4: 写 site.webmanifest 与 browserconfig.xml（经 Write 工具）**

`D:\code\blog\assets\img\favicons\site.webmanifest`：

```liquid
---
layout: compress
---

{% assign favicon_path = "/assets/img/favicons" | relative_url %}

{
  "name": "{{ site.title }}",
  "short_name": "Cheney 博客",
  "description": "{{ site.description | strip_newlines }}",
  "icons": [
    {
      "src": "{{ favicon_path }}/android-chrome-192x192.png",
      "sizes": "192x192",
      "type": "image/png",
      "purpose": "any"
    },
    {
      "src": "{{ favicon_path }}/android-chrome-512x512.png",
      "sizes": "512x512",
      "type": "image/png",
      "purpose": "any"
    },
    {
      "src": "{{ favicon_path }}/android-chrome-512x512.png",
      "sizes": "512x512",
      "type": "image/png",
      "purpose": "maskable"
    }],
  "start_url": "{{ '/index.html' | relative_url }}",
  "theme_color": "#15161a",
  "background_color": "#f7f7f7",
  "display": "standalone"
}
```

`D:\code\blog\assets\img\favicons\browserconfig.xml`：

```xml
---
layout: compress
---

<?xml version="1.0" encoding="utf-8"?>
<browserconfig>
  <msapplication>
    <tile>
      <square150x150logo src="{{ '/assets/img/favicons/mstile-150x150.png' | relative_url }}" />
      <TileColor>#15161a</TileColor>
    </tile>
  </msapplication>
</browserconfig>
```

前置校验（防 JSON 破行）：`grep -c '"' <(grep '^description' /d/code/blog/_config.yml)` 之外更直接——运行 `sed -n '/^description/,/^$/p' /d/code/blog/_config.yml | grep -c '"'`，期望输出 `0`（值里没有双引号；有的话把 manifest 里 description 改写成手工字符串）。

- [ ] **Step 5: 写 `_includes/favicons.html`（站点覆盖）**

`D:\code\blog\_includes\favicons.html`（全量内容）：

```html
<!--
  Site icons (identity layer). Replaces the theme default favicon set.
-->

{% capture favicon_path %}{{ '/assets/img/favicons' | relative_url }}{% endcapture %}

<link rel="icon" type="image/svg+xml" sizes="any" href="{{ favicon_path }}/favicon.svg">
<link rel="apple-touch-icon" sizes="180x180" href="{{ favicon_path }}/apple-touch-icon.png">
<link rel="icon" type="image/png" sizes="32x32" href="{{ favicon_path }}/favicon-32x32.png">
<link rel="icon" type="image/png" sizes="16x16" href="{{ favicon_path }}/favicon-16x16.png">
{% if site.pwa.enabled %}
  <link rel="manifest" href="{{ favicon_path }}/site.webmanifest">
{% endif %}
<link rel="shortcut icon" href="{{ favicon_path }}/favicon.ico">
<meta name="apple-mobile-web-app-title" content="{{ site.title }}">
<meta name="application-name" content="{{ site.title }}">
<meta name="msapplication-TileColor" content="#15161a">
<meta name="msapplication-config" content="{{ favicon_path }}/browserconfig.xml">
<meta name="theme-color" content="#f7f7f7">
```

- [ ] **Step 6: 构建并核对**

Run:
```bash
cd /d/code/blog && bash /d/Ruby33-x64/bin/bundle exec jekyll build 2>&1 | tail -5
ls _site/assets/img/favicons/
grep -o "favicon.svg" _site/index.html | head -1
grep -o "msapplication-TileColor\" content=\"#15161a" _site/index.html
```
Expected: 构建无 error（"done in ..."）；favicons 目录 10 个文件；两处 grep 均有输出。

- [ ] **Step 7: 16px 实尺目检（关键门禁）**

把产出拷到核验目录，写 `favicon-check.html`（`C:/Users/hack9/AppData/Local/Temp/blog-shots/`），用 http://127.0.0.1:4408（该预览服务在跑；若停了下述命令确认后重启：`cd "/c/Users/hack9/AppData/Local/Temp/blog-shots" && (/c/Python314/python -m http.server 4408 > /tmp/http4408.log 2>&1 & echo $! > /tmp/http4408.pid)`）：

```bash
mkdir -p "/c/Users/hack9/AppData/Local/Temp/blog-shots/favicons" && cp D:/code/blog/assets/img/favicons/*.png "/c/Users/hack9/AppData/Local/Temp/blog-shots/favicons/"
```

`favicon-check.html` 内容：

```html
<!DOCTYPE html><html lang="zh-CN"><head><meta charset="utf-8"><title>favicon check</title>
<style>
 body{font-family:Consolas,"Microsoft YaHei",monospace;background:#111;color:#ddd;padding:24px}
 h2{font-size:14px;color:#9cf;margin:18px 0 8px}
 .tabs{display:flex;gap:18px;align-items:center}
 .tab{display:flex;align-items:center;gap:8px;padding:6px 12px;border-radius:8px 8px 0 0;font-size:13px}
 .lt{background:#dee1e6;color:#333}.dt{background:#202124;color:#e8eaed}
 img.p16{width:16px;height:16px}
 img.p32{width:32px;height:32px}
 .big{display:flex;gap:20px;align-items:flex-end}
 .big img{display:block}
</style></head><body>
<h2>A. 16px 真实尺寸（亮/暗标签栏模拟）</h2>
<div class="tabs">
  <div class="tab lt"><img class="p16" src="./favicons/favicon-16x16.png"><span>Cheney 技术博客</span></div>
  <div class="tab dt"><img class="p16" src="./favicons/favicon-16x16.png"><span>Cheney 技术博客</span></div>
</div>
<h2>B. 32px 真实尺寸</h2>
<div class="tabs">
  <div class="tab lt"><img class="p32" src="./favicons/favicon-32x32.png"><span>Cheney 技术博客</span></div>
  <div class="tab dt"><img class="p32" src="./favicons/favicon-32x32.png"><span>Cheney 技术博客</span></div>
</div>
<h2>C. 大尺寸</h2>
<div class="big">
  <img src="./favicons/apple-touch-icon.png" width="180">
  <img src="./favicons/android-chrome-192x192.png" width="192">
  <img src="./favicons/android-chrome-512x512.png" width="128">
</div>
</body></html>
```

截图查看：
```bash
"/c/Program Files/Google/Chrome/Application/chrome.exe" --headless=new --disable-gpu --hide-scrollbars --force-device-scale-factor=2 --window-size=900,620 --virtual-time-budget=6000 --screenshot="C:\Users\hack9\AppData\Local\Temp\blog-shots\favicon-check.png" "http://127.0.0.1:4408/favicon-check.html" 2>/dev/null
```
然后 Read 该 PNG。

**判定标准：** 16px 下能看出「深底 + 亮蓝开口环」即可（细节允许模糊）；若完全糊成一团/看不出开口，把脚本里 `small` 参数从 `[8.4, 5.8, 14]` 调成 `[9.2, 6.6, 12]` 后重跑 Step 2，并重新拷贝核验（最多调两轮）。

- [ ] **Step 8: 提交**

```bash
cd /d/code/blog
git add assets/img/favicons _includes/favicons.html
git status --short
git commit -m "视觉身份：F2 环点标图标套件（SVG/ICO/PWA manifest 与浏览器集成收口）"
git log --oneline -1
```

---

### Task 2: 头像精修（去水印、方形裁切）

**Files:**
- Modify: `assets/images/avatar.jpg`（原地替换，仓库内所有引用自动生效）
- Scratch: `C:/Users/hack9/AppData/Local/Temp/blog-shots/gen-avatar.mjs`

- [ ] **Step 1: 备份原图 + 拷入核验目录**

```bash
cp D:/code/blog/assets/images/avatar.jpg "/c/Users/hack9/AppData/Local/Temp/blog-shots/avatar-src.jpg"
curl -s -o /dev/null -w "%{http_code}\n" http://127.0.0.1:4408/avatar-src.jpg
```
Expected: `200`（预览服务在跑；否则按 Task 1 Step 7 的方式重启 4408）。

- [ ] **Step 2: 写生成脚本 `gen-avatar.mjs` 并运行**

写入 `C:/Users/hack9/AppData/Local/Temp/blog-shots/gen-avatar.mjs`：

```js
// 一次性脚本：裁 (60,0) 起 700×700（去小红书水印，保留「不用管我」与猫），q92 覆写仓库头像
import { setTimeout as sleep } from 'node:timers/promises';
import { spawn } from 'node:child_process';
import fs from 'node:fs';

const PORT = 9337;
const CHROME = 'C:/Program Files/Google/Chrome/Application/chrome.exe';
const TMP = 'C:/Users/hack9/AppData/Local/Temp/blog-shots';

const chrome = spawn(CHROME, [
  '--headless=new', '--disable-gpu', '--no-first-run', '--no-default-browser-check',
  `--remote-debugging-port=${PORT}`,
  '--user-data-dir=C:/Users/hack9/AppData/Local/Temp/chrome-avatar-profile',
  'about:blank',
], { stdio: 'ignore' });

async function waitVersion() {
  for (let i = 0; i < 50; i++) {
    try {
      const r = await fetch(`http://127.0.0.1:${PORT}/json/version`);
      if (r.ok) return;
    } catch {}
    await sleep(300);
  }
  throw new Error('chrome not up');
}
await waitVersion();

const resp = await fetch(`http://127.0.0.1:${PORT}/json/new?about:blank`, { method: 'PUT' });
const target = await resp.json();
const ws = new WebSocket(target.webSocketDebuggerUrl);
let id = 0;
const pend = new Map();
const send = (method, params = {}) =>
  new Promise((res, rej) => {
    const i = ++id;
    pend.set(i, { res, rej });
    ws.send(JSON.stringify({ id: i, method, params }));
  });
await new Promise((r) => (ws.onopen = r));
ws.onmessage = (ev) => {
  const m = JSON.parse(ev.data);
  if (m.id && pend.has(m.id)) {
    const { res, rej } = pend.get(m.id);
    pend.delete(m.id);
    m.error ? rej(new Error(JSON.stringify(m.error))) : res(m.result);
  }
};

await send('Page.enable');
await send('Page.navigate', { url: 'http://127.0.0.1:4408/crop-check.html' });
await sleep(2500);

const expr = `(async () => {
  const img = new Image();
  img.src = '/avatar-src.jpg';
  await img.decode();
  const c = document.createElement('canvas');
  c.width = 700; c.height = 700;
  c.getContext('2d').drawImage(img, 60, 0, 700, 700, 0, 0, 700, 700);
  return c.toDataURL('image/jpeg', 0.92).slice('data:image/jpeg;base64,'.length);
})()`;

const r = await send('Runtime.evaluate', { expression: expr, awaitPromise: true, returnByValue: true });
if (!r.result.value) throw new Error('render failed: ' + JSON.stringify(r));
const buf = Buffer.from(r.result.value, 'base64');
fs.writeFileSync('D:/code/blog/assets/images/avatar.jpg', buf);
fs.writeFileSync(`${TMP}/avatar-fixed.jpg`, buf);
console.log('wrote avatar.jpg', buf.length, 'bytes');

await send('Browser.close').catch(() => {});
process.exit(0);
```

Run:
```bash
cd "/c/Users/hack9/AppData/Local/Temp/blog-shots" && node gen-avatar.mjs
```
Expected: `wrote avatar.jpg` 大小约 40–90KB（若 <20KB 或 >150KB 停下检查）。

- [ ] **Step 3: 渲染目检（112px 圆 + 全幅方）**

`avatar-check.html`（同上目录，引用 `./avatar-fixed.jpg`）：

```html
<!DOCTYPE html><html lang="zh-CN"><head><meta charset="utf-8"><title>avatar check</title>
<style>
 body{font-family:Consolas,"Microsoft YaHei";background:#111;color:#ddd;padding:24px}
 h2{font-size:14px;color:#9cf;margin:18px 0 8px}
 .row{display:flex;gap:24px;align-items:flex-start}
 .panel{padding:14px;border-radius:8px}.dark{background:#1e1e1e}.light{background:#f6f8fa}
 .face{width:112px;height:112px;border-radius:50%;overflow:hidden;display:block}
 .face img{width:100%;height:100%;display:block}
 .full{width:280px;border-radius:8px;display:block}
</style></head><body>
<h2>A. 侧栏 112px 圆形实况（暗/亮底）</h2>
<div class="row">
  <div class="panel dark"><span class="face"><img src="./avatar-fixed.jpg"></span></div>
  <div class="panel light"><span class="face"><img src="./avatar-fixed.jpg"></span></div>
</div>
<h2>B. 全幅方形</h2>
<img class="full" src="./avatar-fixed.jpg">
</body></html>
```

```bash
"/c/Program Files/Google/Chrome/Application/chrome.exe" --headless=new --disable-gpu --hide-scrollbars --force-device-scale-factor=2 --window-size=760,700 --virtual-time-budget=6000 --screenshot="C:\Users\hack9\AppData\Local\Temp\blog-shots\avatar-check.png" "http://127.0.0.1:4408/avatar-check.html" 2>/dev/null
```
Read 截图。
**判定标准：** 圆形内无任何小红书 logo/号码残留；「不用管我」四字完整、猫脸完整；四角无异常裁切。不合格则调整 `drawImage` 的 crop 参数（x=60,y=0,size=700 为基础，y 可上移到 690 增强水印安全性；x 微调 ±10 保构图），重跑 Step 2–3。

- [ ] **Step 4: 提交**

```bash
cd /d/code/blog
git add assets/images/avatar.jpg
git status --short
git commit -m "视觉身份：头像精修（去小红书水印、干净方形裁切，700×700 q92）"
git log --oneline -1
```

---

### Task 3: 身份层（Inter 自托管 + C2 工程蓝 + T2 侧栏标题）

**Files:**
- Create: `assets/fonts/inter/inter-latin-wght-normal.woff2`（下载）
- Create: `assets/css/identity.css`
- Create: `_data/origin/cors.yml`
- Create: `_sass/variables-hook.scss`
- Create: `_includes/sidebar.html`
- Modify: `_config.yml`（`title:` 一行）

- [ ] **Step 1: 下载 Inter 变量字体并校验**

```bash
cd /d/code/blog && mkdir -p assets/fonts/inter && curl -sL -o assets/fonts/inter/inter-latin-wght-normal.woff2 "https://cdn.jsdelivr.net/fontsource/fonts/inter:vf@latest/latin-wght-normal.woff2" && ls -l assets/fonts/inter/ && head -c 4 assets/fonts/inter/inter-latin-wght-normal.woff2
```
Expected: 大小约 30–60KB；`head -c 4` 输出 `wOF2`。若不足 20KB 说明下载失败，重试或改用静态三件套（`inter@latest/latin-400-normal.woff2`、`-600-`、`-700-`，并相应把 identity.css 拆成三条 @font-face）。

- [ ] **Step 2: 写 `assets/css/identity.css`（全量内容，经 Write 工具）**

```css
/* ==========================================================================
   视觉身份覆盖层（Identity Layer）
   加载机制：_data/origin/cors.yml 的 webfonts 槽位（主题 basic.yml 同款用法），
   位置在主题主样式之后，同名覆盖以本文件为准。
   ========================================================================== */

/* --- 1. 自托管 Inter（拉丁字母/数字/符号；中文走系统字体链） --- */
@font-face {
  font-family: 'Inter';
  font-style: normal;
  font-weight: 100 900;
  font-display: swap;
  src: url('../fonts/inter/inter-latin-wght-normal.woff2') format('woff2');
}

/* --- 2. C2 工程蓝 + T2 标题配色（镜像主题四象限：系统偏好 × 手动切换） --- */
@media (prefers-color-scheme: light) {
  html:not([data-mode]),
  html[data-mode='light'] {
    --link-color: #2563eb;
    --link-underline-color: rgba(37, 99, 235, 0.4);
    --toc-highlight: #1d4ed8;
    --checkbox-checked-color: #2563eb;
    --btn-share-hover-color: #2563eb;
    --tag-border: rgba(37, 99, 235, 0.35);
    --tag-hover: rgba(37, 99, 235, 0.12);
    --id-link-hover: #1d4ed8;
    --id-title-main: #202124;
    --id-title-sub: #6b7280;
  }

  html[data-mode='dark'] {
    --link-color: #93c5fd;
    --link-underline-color: rgba(147, 197, 253, 0.4);
    --toc-highlight: #93c5fd;
    --checkbox-checked-color: #93c5fd;
    --btn-share-hover-color: #93c5fd;
    --tag-border: rgba(147, 197, 253, 0.35);
    --tag-hover: rgba(147, 197, 253, 0.14);
    --id-link-hover: #bfdbfe;
    --id-title-main: #e8eaed;
    --id-title-sub: #8a8f96;
  }
}

@media (prefers-color-scheme: dark) {
  html:not([data-mode]),
  html[data-mode='dark'] {
    --link-color: #93c5fd;
    --link-underline-color: rgba(147, 197, 253, 0.4);
    --toc-highlight: #93c5fd;
    --checkbox-checked-color: #93c5fd;
    --btn-share-hover-color: #93c5fd;
    --tag-border: rgba(147, 197, 253, 0.35);
    --tag-hover: rgba(147, 197, 253, 0.14);
    --id-link-hover: #bfdbfe;
    --id-title-main: #e8eaed;
    --id-title-sub: #8a8f96;
  }

  html[data-mode='light'] {
    --link-color: #2563eb;
    --link-underline-color: rgba(37, 99, 235, 0.4);
    --toc-highlight: #1d4ed8;
    --checkbox-checked-color: #2563eb;
    --btn-share-hover-color: #2563eb;
    --tag-border: rgba(37, 99, 235, 0.35);
    --tag-hover: rgba(37, 99, 235, 0.12);
    --id-link-hover: #1d4ed8;
    --id-title-main: #202124;
    --id-title-sub: #6b7280;
  }
}

/* --- 3. 干掉主题硬编码的橙色 hover（%link-hover 编译产物，10 组选择器） --- */
footer a:hover,
#topbar #breadcrumb a:hover,
#page-category a:hover,
#page-tag a:hover,
#access-lastmod a:hover,
.post-tail-wrapper .license-wrapper > a:hover,
.post-tags .post-tag:hover,
.post-meta a:not([class]):hover,
.content a:not(.img-link):hover,
#search-results a:hover {
  color: var(--id-link-hover) !important;
  border-bottom-color: var(--id-link-hover);
}

/* --- 4. 侧栏标题 T2：大字名 + 小字行 --- */
.site-title {
  font-weight: 400;
}

.site-title-main {
  display: block;
  font-size: 1.5rem;
  font-weight: 700;
  letter-spacing: -0.01em;
  line-height: 1.25;
  color: var(--id-title-main);
}

.site-title-sub {
  display: block;
  margin-top: 0.4rem;
  font-size: 0.75rem;
  font-weight: 600;
  letter-spacing: 0.32em;
  line-height: 1.4;
  color: var(--id-title-sub);
}

.site-title a:hover .site-title-main {
  color: var(--sidebar-active-color);
}
```

- [ ] **Step 3: 写 `_data/origin/cors.yml`（全量内容）**

```yaml
# Resource Hints（仅保留 jsdelivr；Google Fonts 已改自托管 Inter，不再预连接）

resource_hints:
  - url: https://cdn.jsdelivr.net
    links:
      - rel: preconnect
      - rel: dns-prefetch

# Web Fonts：指向站内身份层样式（Inter @font-face + 全站视觉覆盖），替代 Google Fonts
webfonts: /assets/css/identity.css

# Libraries

toc:
  css: https://cdn.jsdelivr.net/npm/tocbot@4.27.20/dist/tocbot.min.css
  js: https://cdn.jsdelivr.net/npm/tocbot@4.27.20/dist/tocbot.min.js

fontawesome:
  css: https://cdn.jsdelivr.net/npm/@fortawesome/fontawesome-free@6.5.2/css/all.min.css

search:
  js: https://cdn.jsdelivr.net/npm/simple-jekyll-search@1.10.0/dist/simple-jekyll-search.min.js

mermaid:
  js: https://cdn.jsdelivr.net/npm/mermaid@10.9.0/dist/mermaid.min.js

dayjs:
  js:
    common: https://cdn.jsdelivr.net/npm/dayjs@1.11.11/dayjs.min.js
    locale: https://cdn.jsdelivr.net/npm/dayjs@1.11.11/locale/:LOCALE.min.js
    relativeTime: https://cdn.jsdelivr.net/npm/dayjs@1.11.11/plugin/relativeTime.min.js
    localizedFormat: https://cdn.jsdelivr.net/npm/dayjs@1.11.11/plugin/localizedFormat.min.js

glightbox:
  css: https://cdn.jsdelivr.net/npm/glightbox@3.3.0/dist/css/glightbox.min.css
  js: https://cdn.jsdelivr.net/npm/glightbox@3.3.0/dist/js/glightbox.min.js

lazy-polyfill:
  css: https://cdn.jsdelivr.net/npm/loading-attribute-polyfill@2.1.1/dist/loading-attribute-polyfill.min.css
  js: https://cdn.jsdelivr.net/npm/loading-attribute-polyfill@2.1.1/dist/loading-attribute-polyfill.umd.min.js

clipboard:
  js: https://cdn.jsdelivr.net/npm/clipboard@2.0.11/dist/clipboard.min.js

mathjax:
  js: https://cdn.jsdelivr.net/npm/mathjax@3.2.2/es5/tex-chtml.js
```

- [ ] **Step 4: 写 `_sass/variables-hook.scss`（经 Write 工具，自动建目录）**

```scss
/*
  主题变量钩子：main.scss 在 addon/variables 之后 import 此文件，
  站点版优先于主题内同名空文件，用于覆盖 $font-family-*。
  拉丁字母/数字用自托管 Inter（assets/fonts/inter/），中文回退系统字体链。
*/
$font-family-base: 'Inter', 'PingFang SC', 'HarmonyOS Sans SC', 'Noto Sans SC', 'Microsoft YaHei',
  sans-serif;
$font-family-heading: 'Inter', 'PingFang SC', 'HarmonyOS Sans SC', 'Noto Sans SC', 'Microsoft YaHei',
  sans-serif;
```

- [ ] **Step 5: 写 `_includes/sidebar.html`（站点覆盖，全量内容）**

```html
<!-- The Side Bar -->

{% assign title_parts = site.title | split: ' ' %}
{% assign title_main = title_parts | first %}
{% assign title_sub = title_parts | last %}

<aside aria-label="Sidebar" id="sidebar" class="d-flex flex-column align-items-end">
  <header class="profile-wrapper">
    <a href="{{ '/' | relative_url }}" id="avatar" class="rounded-circle">
      {%- if site.avatar != empty and site.avatar -%}
        {%- capture avatar_url -%}
          {% include media-url.html src=site.avatar %}
        {%- endcapture -%}
        <img src="{{- avatar_url -}}" width="112" height="112" alt="avatar" onerror="this.style.display='none'">
      {%- endif -%}
    </a>

    <h1 class="site-title">
      <a href="{{ '/' | relative_url }}">
        <span class="site-title-main">{{ title_main }}</span>
        <span class="site-title-sub">{{ title_sub }}</span>
      </a>
    </h1>
    <p class="site-subtitle mb-0">{{ site.tagline }}</p>
  </header>
  <!-- .profile-wrapper -->

  <nav class="flex-column flex-grow-1 w-100 ps-0">
    <ul class="nav">
      <!-- home -->
      <li class="nav-item{% if page.layout == 'home' %}{{ " active" }}{% endif %}">
        <a href="{{ '/' | relative_url }}" class="nav-link">
          <i class="fa-fw fas fa-home"></i>
          <span>{{ site.data.locales[include.lang].tabs.home | upcase }}</span>
        </a>
      </li>
      <!-- the real tabs -->
      {% for tab in site.tabs %}
        <li class="nav-item{% if tab.url == page.url %}{{ " active" }}{% endif %}">
          <a href="{{ tab.url | relative_url }}" class="nav-link">
            <i class="fa-fw {{ tab.icon }}"></i>
            {% capture tab_name %}{{ tab.url | split: '/' }}{% endcapture %}

            <span>{{ site.data.locales[include.lang].tabs.[tab_name] | default: tab.title | upcase }}</span>
          </a>
        </li>
        <!-- .nav-item -->
      {% endfor %}
    </ul>
  </nav>

  <div class="sidebar-bottom d-flex flex-wrap  align-items-center w-100">
    {% unless site.theme_mode %}
      <button type="button" class="btn btn-link nav-link" aria-label="Switch Mode" id="mode-toggle">
        <i class="fas fa-adjust"></i>
      </button>

      {% if site.data.contact.size > 0 %}
        <span class="icon-border"></span>
      {% endif %}
    {% endunless %}

    {% for entry in site.data.contact %}
      {% case entry.type %}
        {% when 'github', 'twitter' %}
          {%- capture url -%}
            https://{{ entry.type }}.com/{{ site[entry.type].username }}
          {%- endcapture -%}
        {% when 'email' %}
          {% assign email = site.social.email | split: '@' %}
          {%- capture url -%}
            javascript:location.href = 'mailto:' + ['{{ email[0] }}','{{ email[1] }}'].join('@')
          {%- endcapture -%}
        {% when 'rss' %}
          {% assign url = '/feed.xml' | relative_url %}
        {% else %}
          {% assign url = entry.url %}
      {% endcase %}

      {% if url %}
        <a
          href="{{ url }}"
          aria-label="{{ entry.type }}"
          {% assign link_types = '' %}

          {% unless entry.noblank %}
            target="_blank"
            {% assign link_types = 'noopener noreferrer' %}
          {% endunless %}

          {% if entry.type == 'mastodon' %}
            {% assign link_types = link_types | append: ' me' | strip %}
          {% endif %}

          {% unless link_types == empty %}
            rel="{{ link_types }}"
          {% endunless %}
        >
          <i class="{{ entry.icon }}"></i>
        </a>
      {% endif %}
    {% endfor %}
  </div>
  <!-- .sidebar-bottom -->
</aside>
<!-- #sidebar -->
```

- [ ] **Step 6: 改 `_config.yml` 站点名**

`D:\code\blog\_config.yml` 第 17 行：`title: Cheney 的技术博客 # the main title` → 改为：

```yaml
title: Cheney 技术博客 # the main title
```

- [ ] **Step 7: 构建并做静态核验**

Run:
```bash
cd /d/code/blog && bash /d/Ruby33-x64/bin/bundle exec jekyll build 2>&1 | tail -5
echo "--- googleapis 残留（期望 0 处）:"; grep -rl "fonts.googleapis.com" _site --include='*.html'; echo "--- identity.css 链接:"; grep -o 'href="[^"]*identity.css"' _site/index.html
echo "--- 编译进主题 CSS 的 Inter:"; grep -c "Inter" _site/assets/css/jekyll-theme-chirpy.css
echo "--- identity.css 字体路径:"; grep -o "fonts/inter/inter-latin-wght-normal.woff2" _site/assets/css/identity.css
echo "--- 侧栏两行:"; grep -o 'site-title-main">[^<]*' _site/index.html; grep -o 'site-title-sub">[^<]*' _site/index.html
```
Expected: 构建成功；googleapis 无输出；identity.css 链接存在；Inter 计数 ≥1；字体路径 1 处；两行分别输出 `site-title-main">Cheney` 与 `site-title-sub">技术博客`。
若 `site-title-sub` 为空 → 检查 `_config.yml` 的 title 是否已改为空格分隔两段。

- [ ] **Step 8: 提交**

```bash
cd /d/code/blog
git add assets/fonts assets/css/identity.css _data/origin/cors.yml _sass/variables-hook.scss _includes/sidebar.html _config.yml
git status --short
git commit -m "视觉身份：T2 侧栏标题、C2 工程蓝、Inter 自托管字体与身份覆盖层（去 Google Fonts）"
git log --oneline -1
```
注意：`git status --short` 里应**不包含** `_tabs/friends.md` 与 `assets/css/jekyll-theme-chirpy.scss`（并行会话改动，保持未暂存）。

---

### Task 4: 全站双主题视觉核验（门禁）

**Files:**
- Scratch: `C:/Users/hack9/AppData/Local/Temp/blog-shots/verify-identity.mjs`
- 可能修复：`assets/css/identity.css`（样式微调）、`assets/img/favicons/*`（图标校准）、`_includes/*`

- [ ] **Step 1: 起 _site 预览服务（带 baseurl 的正确路径）**

```bash
rm -rf "/c/Users/hack9/AppData/Local/Temp/site-preview" && mkdir -p "/c/Users/hack9/AppData/Local/Temp/site-preview" && cp -r /d/code/blog/_site "/c/Users/hack9/AppData/Local/Temp/site-preview/blog"
cd "/c/Users/hack9/AppData/Local/Temp/site-preview" && (/c/Python314/python -m http.server 4409 > /tmp/http4409.log 2>&1 & echo $! > /tmp/http4409.pid) && sleep 1 && curl -s -o /dev/null -w "%{http_code}\n" http://127.0.0.1:4409/blog/
```
Expected: `200`（路径为 `/blog/`，与 baseurl 对应）。

- [ ] **Step 2: 写核验脚本 `verify-identity.mjs`（全量内容）**

```js
// 双主题核验：变量值、标题排版、字体、hover 颜色 + 关键区域截图
import { setTimeout as sleep } from 'node:timers/promises';
import { spawn } from 'node:child_process';
import fs from 'node:fs';

const PORT = 9338;
const CHROME = 'C:/Program Files/Google/Chrome/Application/chrome.exe';
const OUT = 'C:/Users/hack9/AppData/Local/Temp/blog-shots';
const BASE = 'http://127.0.0.1:4409/blog/';

const chrome = spawn(CHROME, [
  '--headless=new', '--disable-gpu', '--hide-scrollbars', '--no-first-run', '--no-default-browser-check',
  `--remote-debugging-port=${PORT}`,
  '--user-data-dir=C:/Users/hack9/AppData/Local/Temp/chrome-verify-profile',
  'about:blank',
], { stdio: 'ignore' });

async function waitVersion() {
  for (let i = 0; i < 50; i++) {
    try {
      const r = await fetch(`http://127.0.0.1:${PORT}/json/version`);
      if (r.ok) return;
    } catch {}
    await sleep(300);
  }
  throw new Error('chrome not up');
}
await waitVersion();

const resp = await fetch(`http://127.0.0.1:${PORT}/json/new?about:blank`, { method: 'PUT' });
const target = await resp.json();
const ws = new WebSocket(target.webSocketDebuggerUrl);
let id = 0;
const pend = new Map();
const send = (method, params = {}) =>
  new Promise((res, rej) => {
    const i = ++id;
    pend.set(i, { res, rej });
    ws.send(JSON.stringify({ id: i, method, params }));
  });
await new Promise((r) => (ws.onopen = r));
ws.onmessage = (ev) => {
  const m = JSON.parse(ev.data);
  if (m.id && pend.has(m.id)) {
    const { res, rej } = pend.get(m.id);
    pend.delete(m.id);
    m.error ? rej(new Error(JSON.stringify(m.error))) : res(m.result);
  }
};

const evalJs = async (expression, awaitPromise = false) => {
  const r = await send('Runtime.evaluate', { expression, awaitPromise, returnByValue: true });
  return r.result.value;
};

await send('Page.enable');
await send('Emulation.setDeviceMetricsOverride', { width: 1200, height: 900, deviceScaleFactor: 2, mobile: false });

async function runTheme(theme) {
  await send('Emulation.setEmulatedMedia', { features: [{ name: 'prefers-color-scheme', value: theme }] });
  await send('Page.navigate', { url: BASE });
  await sleep(3500);

  const checks = await evalJs(`(async () => {
    await document.fonts.ready;
    const root = getComputedStyle(document.documentElement);
    const main = document.querySelector('.site-title-main');
    const sub = document.querySelector('.site-title-sub');
    const tagline = document.querySelector('.site-subtitle');
    const cs = main ? getComputedStyle(main) : null;
    const scs = sub ? getComputedStyle(sub) : null;
    return JSON.stringify({
      linkColor: root.getPropertyValue('--link-color').trim(),
      linkHover: root.getPropertyValue('--id-link-hover').trim(),
      tocHighlight: root.getPropertyValue('--toc-highlight').trim(),
      titleMain: cs ? { text: main.textContent, size: cs.fontSize, weight: cs.fontWeight, color: cs.color, family: cs.fontFamily.split(',')[0] } : null,
      titleSub: scs ? { text: sub.textContent, size: scs.fontSize, spacing: scs.letterSpacing, color: scs.color } : null,
      taglineStyle: tagline ? getComputedStyle(tagline).fontStyle : null,
      interLoaded: document.fonts.check('16px Inter'),
      bodyFamily: getComputedStyle(document.body).fontFamily.split(',')[0],
    }, null, 1);
  })()`, true);
  console.log(`== ${theme} ==\n${checks}`);

  const s1 = await send('Page.captureScreenshot', { format: 'png', clip: { x: 0, y: 0, width: 260, height: 460, scale: 2 } });
  fs.writeFileSync(`${OUT}/id-sidebar-${theme}.png`, Buffer.from(s1.data, 'base64'));

  // 进一篇文章页做正文 hover / 目录 / 标签核验
  const postUrl = await evalJs(`[...document.querySelectorAll('a')].map(a => a.href).find(h => h.includes('/posts/')) || ''`);
  if (postUrl) {
    await send('Page.navigate', { url: postUrl });
    await sleep(3500);
    const rect = await evalJs(`(() => {
      const a = document.querySelector('.content a:not(.img-link)');
      if (!a) return null;
      const r = a.getBoundingClientRect();
      return JSON.stringify({ x: r.x + r.width / 2, y: r.y + r.height / 2 });
    })()`);
    if (rect) {
      const { x, y } = JSON.parse(rect);
      await send('Input.dispatchMouseEvent', { type: 'mouseMoved', x, y });
      await sleep(600);
      const hover = await evalJs(`(() => {
        const a = document.querySelector('.content a:not(.img-link):hover');
        if (!a) return 'no hover';
        const c = getComputedStyle(a);
        return JSON.stringify({ color: c.color, borderBottomColor: c.borderBottomColor });
      })()`);
      console.log(`${theme} hover:`, hover);
    }
    const s2 = await send('Page.captureScreenshot', { format: 'png', clip: { x: 0, y: 0, width: 1200, height: 900, scale: 1 } });
    fs.writeFileSync(`${OUT}/id-post-${theme}.png`, Buffer.from(s2.data, 'base64'));
  }
}

await runTheme('dark');
await runTheme('light');

await send('Browser.close').catch(() => {});
process.exit(0);
```

Run:
```bash
cd "/c/Users/hack9/AppData/Local/Temp/blog-shots" && node verify-identity.mjs
```
Expected（暗色）：`linkColor: #93c5fd`、`linkHover: #bfdbfe`、`titleMain: { text: "Cheney", size: "24px", weight: "700", color: "rgb(232, 234, 237)" }`、`titleSub.spacing: "3.84px"`、`taglineStyle: "normal"`、`interLoaded: true`、hover color `rgb(191, 219, 254)`；亮色对应 `#2563eb` / `#1d4ed8` / `rgb(32, 33, 36)`，hover `rgb(29, 78, 216)`。

- [ ] **Step 3: Read 四张截图判定**

依次 Read：`id-sidebar-dark.png`、`id-sidebar-light.png`、`id-post-dark.png`、`id-post-light.png`。
**判定：** 侧栏两行标题层级清晰、无 900 伪粗、tagline 非斜体；文章页目录高亮/标签/链接为蓝调；整体无橙色残留、无布局破坏。发现偏差 → 微调 `identity.css` 对应值（不要动版式选择器）→ 回 Step 2 重跑（增量看截图即可）。

- [ ] **Step 4: 回归自查（对照 spec §4 验收标准）**

```bash
cd /d/code/blog
grep -o "series-nav" _site/assets/js/*.js 2>/dev/null | head -1   # 系列导航脚本仍在
grep -c "giscus" _site/posts/*/index.html 2>/dev/null | grep -v ":0" | head -3   # 评论挂载仍在
grep -o "goatcounter" _site/index.html | head -1
grep -c "feed.xml" _site/index.html
```
Expected: 各项均有输出（功能文件未被本次改动触碰，属"未见异常即通过"型检查）。

- [ ] **Step 5: 提交修复（如有）+ 清点**

若 Step 3 有修复则：
```bash
git add assets/css/identity.css   # 或其他被改文件，按实际显式路径
git commit -m "视觉身份：核验后样式微调"
```
最后 `git log --oneline -6` 确认本计划产生的提交链（Task1–4）。

---

### Task 5: 汇报、push 门禁与线上核验

**Files:** 无代码改动（部署与验证）

- [ ] **Step 1: 向用户汇报**

汇总：改动清单（图标/头像/标题/强调色/字体）、四张核验截图要点、spec §4 验收对照结果。**请求 push 许可**（push 触发 Pages 部署，必须显式确认）。

- [ ] **Step 2: push（用户确认后）**

```bash
cd /d/code/blog && git push </dev/null
```
注：沙箱下若报 `cannot create standard input pipe`，`</dev/null` 重定向即可绕过。

- [ ] **Step 3: 线上核验（部署 run 成功后）**

```bash
sleep 60
curl -s -o /dev/null -w "favicon.svg %{http_code} %{content_type}\n" "https://cheneyzhang93.github.io/blog/assets/img/favicons/favicon.svg"
curl -s -o /dev/null -w "identity.css %{http_code}\n" "https://cheneyzhang93.github.io/blog/assets/css/identity.css"
curl -s -o /dev/null -w "woff2 %{http_code}\n" "https://cheneyzhang93.github.io/blog/assets/fonts/inter/inter-latin-wght-normal.woff2"
curl -s "https://cheneyzhang93.github.io/blog/" | grep -o 'href="[^"]*identity.css"' | head -1
curl -s "https://cheneyzhang93.github.io/blog/" | grep -c "fonts.googleapis.com"
```
Expected: 三个 200 + 正确 content-type；HTML 含 identity.css；googleapis 计数 0。
再跑一次 `verify-identity.mjs`（把 `BASE` 临时改成线上 URL）做线上双主题终检；favicon 因浏览器强缓存，线上目检用无痕窗口或新 profile。

- [ ] **Step 4: 更新记忆文件**

更新 `D:\QoderData\.qoder-cn\projects\d--code-blog\memory\` 中相关条目（新增「视觉身份层机制」记忆：identity.css 经 webfonts 槽位加载、variables-hook 字体栈、favicon 套件生成方式、orange hover 覆盖清单），并在 `MEMORY.md` 索引中挂一行。

---

## 自审记录（spec → plan 覆盖对照）

| spec 要求 | 对应任务 |
|---|---|
| §2.1 F2 图标套件（含光学校准/manifest/browserconfig/favicons.html） | Task 1 |
| §2.2 T2 标题 + tagline 去斜体 + title 改名 | Task 3 Step 5–6 |
| §2.3 C2 变量 + 去橙 hover（10 组） | Task 3 Step 2 |
| §2.4 Inter 自托管 + 去 Google Fonts（cors.yml/字体栈） | Task 3 Step 1–4 |
| §2.5 头像精修 | Task 2 |
| §4 验收标准（构建/双主题/16px/无回归） | Task 4 |
| §5 风险（favicon 缓存、cors.yml 升级同步、push 门禁） | Task 5 |
| 并行会话文件不触碰 | 全计划以新建文件为主；Task 3 Step 8 有显式核验提示 |
