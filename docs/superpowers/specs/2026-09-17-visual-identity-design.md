# 视觉身份升级设计（Visual Identity Design）

> 日期：2026-09-17 · 状态：已确认（用户选定 T2 / F2 / C2 / 头像精修）· 前置：`docs/superpowers/specs/2026-09-17-tech-influence-plan-design.md`（基建已上线）

## 1. 背景与目标

站点基建（系列导航、giscus、身份体系、SEO）已上线，但视觉仍是主题默认资产：浏览器图标是主题自带的卡通蚂蚁、侧栏标题为 28px/900 灰字（中文伪粗发糊、折两行）、全站「蓝链接 + 橙 hover」杂色、Google Fonts 在国内构成 render-blocking。本次做**身份层升级**：图标套件 + 标题排版 + 强调色 + 中文字体策略 + 头像精修，全部以覆盖层实现，**不动版式结构与任何功能**。

选型结论（已评审）：
- **方案 A：留在 Chirpy 做身份层升级**（否掉换主题：迁移需重做系列导航/giscus/GoatCounter/身份体系/SEO/封面/部署，2–5 个工作日等效 + URL 索引风险；「丑」的来源是默认资产未换，任何主题同理）。
- 视觉气质：**简洁工程感**（Stripe/Linear 式克制，限定单强调色）；头像**保留猫猫，仅修边幅**。
- 变体选择：**T2** 大字名 + 小字行 · **F2** 圆环 + 节点图标 · **C2** 工程蓝 · 头像精修（含去水印）。

## 2. 设计详情

### 2.1 浏览器图标 — F2 环点标（C2 蓝）

- 图形：`#15161a` 深色圆角底（rx≈22%）+ 开口圆环 + 节点圆点，环与点 `#60A5FA`；开口朝右（±40°），节点落在开口中。
- 几何（viewBox 64）：圆心 (32,32) r=16，弧 `M44.26 21.72 A16 16 0 1 0 44.26 42.28`，节点 (48,32)。
- 光学校准（小尺寸单独加粗）：≤32px 用描边 8.4 / 节点 5.8；≥48px 用 6.4 / 4.6；SVG 用中间值 7.0 / 5.0。
- 产出（`assets/img/favicons/`，覆盖主题同名文件）：
  | 文件 | 规格 |
  |---|---|
  | `favicon.svg` | 矢量，现代浏览器直用 |
  | `favicon-16x16.png` / `favicon-32x32.png` | 小尺寸光学校准版 |
  | `favicon.ico` | 内装 16+32+48 |
  | `apple-touch-icon.png` | 180，满幅方底（Apple 自行遮罩） |
  | `android-chrome-192x192.png` / `-512x512.png` | 圆角底；512 同时作 maskable（图形居中 50%，在安全区内） |
  | `mstile-150x150.png` | 满幅方底 |
  | `site.webmanifest` | name=site.title、short_name=`Cheney 博客`、theme_color `#15161a`、background `#f7f7f7`、display **standalone**（改掉主题的 fullscreen） |
  | `browserconfig.xml` | TileColor `#15161a`（替换主题残留橙 `#da532c`） |
- `_includes/favicons.html` 站点覆盖：新增 `<link rel="icon" type="image/svg+xml" sizes="any">`，`msapplication-TileColor` → `#15161a`，`theme-color` → `#f7f7f7`；其余与主题一致。

### 2.2 侧栏标题 — T2 大字名 + 小字行

- 结构（`_includes/sidebar.html` 站点覆盖）：`h1.site-title > a > span.site-title-main + span.site-title-sub`；拆行数据来自 `site.title` 按空格切分（`| first` / `| last`）。
- 规格：
  | 元素 | 规格 |
  |---|---|
  | 第一行（Cheney） | 1.5rem / 700 / letter-spacing -0.01em · 亮 `#202124` · 暗 `#e8eaed` |
  | 第二行（技术博客） | 0.75rem / 600 / letter-spacing 0.32em · 亮 `#6b7280` · 暗 `#8a8f96` |
  | tagline | 去斜体（模板去掉 `fst-italic`），字号与灰色保持主题默认 |
- 悬停：标题悬停时第一行变为 `--sidebar-active-color`（保留主题的侧栏交互感）。
- 配置联动：`_config.yml` 的 `title` 从「Cheney 的技术博客」改为「**Cheney 技术博客**」（浏览器标签、RSS、manifest 同步；侧栏两行由单一来源切分，不硬编码）。

### 2.3 强调色 — C2 工程蓝

覆盖主题变量（含四象限主题结构：跟随系统 × 手动切换）：

| 变量 | 亮色 | 暗色 | 覆盖对象 |
|---|---|---|---|
| `--link-color` | `#2563eb` | `#93c5fd` | 全局链接 |
| `--link-underline-color` | `rgba(37,99,235,.4)` | `rgba(147,197,253,.4)` | 链接下划线 |
| `--toc-highlight` | `#1d4ed8` | `#93c5fd` | 目录当前项 |
| `--checkbox-checked-color` | `#2563eb` | `#93c5fd` | 复选框选中 |
| `--btn-share-hover-color` | `#2563eb` | `#93c5fd` | 分享按钮 hover |
| `--tag-border` | `rgba(37,99,235,.35)` | `rgba(147,197,253,.35)` | 标签 chip 边框 |
| `--tag-hover` | `rgba(37,99,235,.12)` | `rgba(147,197,253,.14)` | 标签 chip hover 底 |
| 自定义 `--id-link-hover` | `#1d4ed8` | `#bfdbfe` | 悬停文字色 |

**干掉主题硬编码橙 `#d2603a`**（`%link-hover` 编译产物，8 组选择器）：页脚、面包屑、分类/标签页、最近更新、文章许可、文内标签、文内 Meta 链接、正文链接、搜索结果——统一 `color: var(--id-link-hover) !important` + `border-bottom-color` 同步。

### 2.4 中文字体策略 — 自托管 Inter + 系统中文链

- 拉丁字母/数字/符号：**Inter**，自托管单文件变量字体（`inter:vf@latest/latin-wght-normal.woff2`，约 45KB，weight 100–900），`@font-face` 声明 `font-display: swap`。
- 中文：系统链 `'PingFang SC', 'HarmonyOS Sans SC', 'Noto Sans SC', 'Microsoft YaHei'` 不变；标题字重不再请求 900（消除雅黑伪粗）。
- 字体栈声明走主题的**变量钩子** `_sass/variables-hook.scss`（主题 main.scss 在其变量声明后 import 此文件——站点版优先，零接触 `assets/css/jekyll-theme-chirpy.scss`，避开并行会话的未提交改动）。
- 资源加载：站点覆盖 `_data/origin/cors.yml`，`webfonts` 槽位指向 `/assets/css/identity.css`（主题 basic.yml 同款机制：CSS 文件经该槽位加载，位置在主题主样式**之后**）；同时移除 `fonts.googleapis.com` / `fonts.gstatic.com` 两条 resource hints。**效果：全站不再请求 Google Fonts**（国内首屏 render-blocking 消除）。
- 说明：`jekyll-theme-chirpy.scss` 里旧字体栈成为失效配置（被钩子覆盖，无副作用）；待并行会话提交后可作为清理项移除。

### 2.5 头像精修

- 原图 820×812 → 裁切 `(60, 0)` 起 700×700（已用渲染验证：小红书 logo 块与号码整体出框，「不用管我」与猫完整保留）→ JPEG q92 覆写 `assets/images/avatar.jpg`。
- 侧栏（112px 圆）、友链页「本站信息」（3.5rem 圆）等所有引用同步生效；侧栏 hover 放大微交互保留。

## 3. 文件清单

**新增**（16 个文件）：
1. `assets/img/favicons/`：favicon.svg / 16 / 32 / ico / apple-180 / android-192 / android-512 / mstile-150 / site.webmanifest / browserconfig.xml
2. `assets/fonts/inter/inter-latin-wght-normal.woff2`
3. `assets/css/identity.css`（@font-face + C2/T2 覆盖层，经 webfonts 槽位加载）
4. `_data/origin/cors.yml`（本地化 webfonts + 去 Google hints；其余键逐字复制）
5. `_sass/variables-hook.scss`（Inter 字体栈）
6. `_includes/sidebar.html`（T2 结构）
7. `_includes/favicons.html`（新图标集）

**修改**（2）：
8. `_config.yml`：`title` → `Cheney 技术博客`
9. `assets/images/avatar.jpg`：精修覆写

**不触碰**：`assets/css/jekyll-theme-chirpy.scss`、`_tabs/friends.md`（并行会话未提交）；`_tabs/friends.md` 中「本站信息」的旧称名与 About 页文案如需统一，留待并行会话提交后处理。

## 4. 验收标准

1. 构建零错误；`_site` 含全部新资产；HTML 中不再出现 `fonts.googleapis.com`；`<head>` 含 `identity.css` 与 `favicon.svg` 链接。
2. 亮/暗双主题：侧栏两行标题、tagline 非斜体、链接/目录/标签为 C2 蓝、无橙色 hover（实际悬停截图验证）。
3. favicon 16px 实尺寸可辨识（渲染目检）；ICO/manifest/browserconfig 内容正确。
4. 拉丁字母渲染为 Inter；中文标题无伪粗。
5. 无功能回归：系列导航、giscus、搜索、GoatCounter、RSS 不受影响。
6. 头像无水印残留。

## 5. 风险与回滚

- 浏览器对 favicon 有强缓存：线上核验需用无痕/新配置文件，旧图标可能滞后显示。
- `_data/origin/cors.yml` 为全量覆盖：主题升级时需对照新版同步（已在文件内注释说明）。
- 回滚：全部为新增文件 + 两处最小修改，`git revert` 对应提交即可整层撤销。
- 部署纪律：本地构建 + 无头视觉核验通过 → 用户确认 → 才 push（触发 Pages 部署）。
