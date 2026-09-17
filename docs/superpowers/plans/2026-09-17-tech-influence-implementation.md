# 技术影响力方案 · 实施计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 落地 spec `docs/superpowers/specs/2026-09-17-tech-influence-plan-design.md` 中不依赖外部账号的站内项（身份收口、系列化、产出物、封面），并为依赖用户参数的接线项（giscus、GSC/Bing）预留精确改动模板。

**Architecture:** 全部改动为站点侧增量：`_config.yml` 配置化、`_includes/` 新增两个组件（metadata-hook、series-nav）、`_posts/*` 仅加 front matter 字段、About 页内容重构。每步以"构建产物 grep 断言"作为测试（沿用 CI 的 html-proofer 作为终检）。

**Tech Stack:** Jekyll 4.3.3 · jekyll-theme-chirpy 7.0.1（锁定）· Git Bash（Windows 构建封装：`bash /d/Ruby33-x64/bin/bundle exec ...`）· html-proofer 5.0.9

**执行纪律（本仓库特有）：**
- 本仓库常有并行会话：每任务开始前 `git status --short` 确认目标文件不在未提交列表；提交一律按明确路径 `git add <files>`；不碰他人未提交文件。
- 本地构建命令（Git Bash，勿用裸 `bundle`，会命中 WindowsApps stub）：`bash /d/Ruby33-x64/bin/bundle exec jekyll build`；production 模式加前缀 `JEKYLL_ENV=production`。
- 统计脚本/PWA 仅 production 构建注入（`js-selector.html:93`），核验这两类需 production 构建。

---

### Task 1: 身份收口（机读配置与署名链接）

**Files:**
- Modify: `_config.yml`（author 改映射、social.links 填 About）
- Modify: `index.html`（seo.links → GitHub）
- Modify: `_tabs/about.md`（seo front matter → ProfilePage + GitHub）
- Create: `_includes/metadata-hook.html`

- [ ] **Step 1: `_config.yml` — author 由字符串升级为映射**

将：
```yaml
# The site author, used by jekyll-seo-tag (`author` meta & JSON-LD).
author: Cheney
```
改为：
```yaml
# The site author, used by jekyll-seo-tag (`author` meta & JSON-LD).
author:
  name: Cheney
  url: "https://cheneyzhang93.github.io/blog/about/"
```

- [ ] **Step 2: `_config.yml` — social.links 替换为 About 链接**

将整段（含 4 条注释候选）：
```yaml
  links:
    # The first element serves as the copyright owner's link
    # - https://x.com/yuchenzhan81879 # change to your twitter homepage
    # - https://github.com/cheneyzhang93 # change to your github homepage
    # Uncomment below to add more social links
    # - https://www.facebook.com/username
    # - https://www.linkedin.com/in/username
```
改为：
```yaml
  links:
    # The first element serves as the copyright owner's link（署名/页脚指向关于页）
    - https://cheneyzhang93.github.io/blog/about/
```

- [ ] **Step 3: 新建 `_includes/metadata-hook.html`**

```liquid
{%- assign github_url = "https://github.com/cheneyzhang93" -%}
<link rel="me" href="{{ github_url }}">

{%- unless page.layout == 'post' %}
  {%- assign person_url = '/about/' | absolute_url %}
  <script type="application/ld+json">
    {
      "@context": "https://schema.org",
      "@type": "Person",
      "@id": "{{ person_url }}#person",
      "name": "{{ site.author.name }}",
      "url": "{{ person_url }}",
      "image": "{{ site.avatar | absolute_url }}",
      "sameAs": ["{{ github_url }}"]
    }
  </script>
{%- endunless %}
```

- [ ] **Step 4: `index.html` front matter 补 seo**

将：
```html
---
layout: home
# Index page
---
```
改为：
```html
---
layout: home
# Index page
seo:
  links:
    - https://github.com/cheneyzhang93
---
```

- [ ] **Step 5: `_tabs/about.md` front matter 补 seo**

在 `order: 4` 之后、`---` 之前插入：
```yaml
seo:
  type: ProfilePage
  links:
    - https://github.com/cheneyzhang93
```

- [ ] **Step 6: 构建**

Run: `bash /d/Ruby33-x64/bin/bundle exec jekyll build`
Expected: `done in ... seconds`，无 error（SCSS deprecation 警告为既有噪音）

- [ ] **Step 7: 断言机读链路（grep 构建产物）**

```bash
grep -o 'href="https://cheneyzhang93.github.io/blog/about/">Cheney</a>' _site/posts/alerting-engine/index.html | head -2
grep -o '<meta name="author" content="Cheney">' _site/index.html
grep -o '"author":{"@type":"Person"[^}]*}' _site/posts/alerting-engine/index.html
grep -o '"@type":"WebSite"' _site/index.html
grep -o '"@type":"ProfilePage"' _site/about/index.html
grep -o '<link rel="me" href="https://github.com/cheneyzhang93">' _site/index.html _site/about/index.html _site/posts/alerting-engine/index.html
grep -c 'about/#person' _site/posts/alerting-engine/index.html || true
grep -c 'about/#person' _site/about/index.html
grep -c 'twitter:site=\|twitter:creator' _site/index.html || true
```
Expected:
- 署名/页脚链接 ≥1 处（byline + footer）；
- author meta 存在；BlogPosting author 含 `"url":"https://cheneyzhang93.github.io/blog/about/"`；
- 首页 `WebSite`；关于页 `ProfilePage`；
- `rel="me"` 三个页面各 1 处；
- `#person` 在文章页 0 次、关于页 1 次；
- twitter:site/creator 0 次（无回归）。

- [ ] **Step 8: 提交**

```bash
git add _config.yml index.html _tabs/about.md _includes/metadata-hook.html
git commit -m "身份链接体系收口：署名链接、作者实体 url、rel=me 与站级 Person 实体"
```

---

### Task 2: 关于页人读链路（GitHub 入口 + 系列导览）

**Files:**
- Modify: `_tabs/about.md`（正文：联系区加 GitHub 行；「从这里读起」重构为两条系列线）

- [ ] **Step 1: 联系与订阅区加 GitHub 行**

在 `## 联系与订阅` 的邮箱行**之前**插入：
```markdown
- **GitHub**：[cheneyzhang93](https://github.com/cheneyzhang93)——代码与实践的补充；
```

- [ ] **Step 2: 「从这里读起」整段替换**

将现有 4 条列表（从 `## 从这里读起` 到该段结束）替换为：
```markdown
## 从这里读起

**稳定性治理（五部曲，按序读）**：

1. [《从零建立测试体系：分层、基座、断言与回归纪律》]({{ '/posts/test-system-from-scratch/' | relative_url }})——223 个用例的从零建设与三笔学费；
2. [《API 契约治理：OpenAPI 3 落地、契约守护与生效边界》]({{ '/posts/api-contract-governance/' | relative_url }})——接口变更「改了就拦得住」；
3. [《日志治理：统一异常出口、结构化日志与字段契约》]({{ '/posts/structured-logging/' | relative_url }})——可观测三件套之一；
4. [《链路追踪治理：W3C 标准落地、上下文传播与 Agent 取舍》]({{ '/posts/distributed-tracing/' | relative_url }})——可观测三件套之二；
5. [《告警治理：事件引擎、通道自建与慢 SQL 感知》]({{ '/posts/alerting-engine/' | relative_url }})——可观测三件套之三。

**后端架构（工程任务复盘）**：

- [《收件箱模式：三方推送消息的可靠消费设计与实践》]({{ '/posts/reliable-message-consume/' | relative_url }})——落库即应答与四态状态机；
- [《批量数据修复：文件账本驱动的可回滚设计》]({{ '/posts/batch-data-fix-file-ledger/' | relative_url }})——文件账本、幂等与回滚。
```

- [ ] **Step 3: 构建并断言**

Run: `bash /d/Ruby33-x64/bin/bundle exec jekyll build`
```bash
grep -o 'href="/blog/posts/[a-z-]*/"' _site/about/index.html | sort -u
```
Expected: 出现全部 7 个文章链接（reliable-message-consume / batch-data-fix-file-ledger / test-system-from-scratch / api-contract-governance / structured-logging / distributed-tracing / alerting-engine）+ friends/feed 链接不受影响。

- [ ] **Step 4: 提交**

```bash
git add _tabs/about.md
git commit -m "关于页：补 GitHub 入口，阅读导览改为两条系列线（七篇全收录）"
```

---

### Task 3: 文章 series front matter（7 篇）

**Files:**
- Modify: `_posts/2026-08-06-reliable-message-consume.md`（+`series: 后端架构`）
- Modify: `_posts/2026-08-13-batch-data-fix-file-ledger.md`（+`series: 后端架构`）
- Modify: `_posts/2026-08-20-test-system-from-scratch.md`（+`series: 稳定性治理`）
- Modify: `_posts/2026-08-27-api-contract-governance.md`（+`series: 稳定性治理`）
- Modify: `_posts/2026-09-03-structured-logging.md`（+`series: 稳定性治理`）
- Modify: `_posts/2026-09-10-distributed-tracing.md`（+`series: 稳定性治理`）
- Modify: `_posts/2026-09-17-alerting-engine.md`（+`series: 稳定性治理`）

- [ ] **Step 1: 每篇在 `categories:` 行之后插入一行 `series: <值>`**

示例（alerting）：
```yaml
categories: [稳定性治理]
series: 稳定性治理
tags: [Java, Spring Boot, 告警引擎, 去重聚合, 慢 SQL, Micrometer]
```
（其余 6 篇同理，仅 `series` 值与文件对应：前两篇 `后端架构`，后五篇 `稳定性治理`；正文与其它 front matter 不动）

- [ ] **Step 2: 断言**

```bash
grep -l "^series:" _posts/*.md | wc -l
grep -h "^series:" _posts/*.md | sort | uniq -c
```
Expected: `7`；`2 后端架构`、`5 稳定性治理`。

- [ ] **Step 3: 提交**

```bash
git add _posts/2026-08-06-reliable-message-consume.md _posts/2026-08-13-batch-data-fix-file-ledger.md _posts/2026-08-20-test-system-from-scratch.md _posts/2026-08-27-api-contract-governance.md _posts/2026-09-03-structured-logging.md _posts/2026-09-10-distributed-tracing.md _posts/2026-09-17-alerting-engine.md
git commit -m "文章 front matter 补 series 字段：稳定性治理 ×5、后端架构 ×2"
```

---

### Task 4: 系列导航组件（series-nav）

**Files:**
- Create: `_includes/series-nav.html`
- Modify: `_layouts/post.html`（tail_includes 首位加 series-nav）

- [ ] **Step 1: 先确认渲染机制**

Run: `grep -n "tail_includes" ~/.local/share/gem/ruby/3.3.0/gems/jekyll-theme-chirpy-7.0.1/_layouts/default.html`
Expected: 确认 default 布局消费 `page.tail_includes` 并 include `<name>.html`。若机制不同，改为在 `_layouts/post.html` 的 `.post-tail-wrapper` 之后直接 `{% include series-nav.html %}`（兜底路径）。

- [ ] **Step 2: 新建 `_includes/series-nav.html`**

```liquid
{% if page.series %}
  {% assign series_posts = site.posts | where: "series", page.series | sort: "date" %}
  {% assign series_total = series_posts | size %}
  {% if series_total > 1 %}
    {% for p in series_posts %}
      {% if p.url == page.url %}
        {% assign series_pos = forloop.index %}
      {% endif %}
    {% endfor %}
    <div class="series-nav text-muted small mt-4 mb-2">
      <div class="mb-1">
        <i class="fas fa-book-open fa-fw me-1"></i>
        本系列《{{ page.series }}》第 {{ series_pos }}/{{ series_total }} 篇 ·
        <a href="{{ site.baseurl }}/categories/{{ page.series | slugify | url_encode }}/">系列目录</a>
      </div>
      <div>
        {% if series_pos > 1 %}
          <a href="{{ series_posts[series_pos | minus: 2].url | relative_url }}">← 上一篇</a>
        {% endif %}
        {% if series_pos < series_total %}
          <a href="{{ series_posts[series_pos].url | relative_url }}" class="ms-3">下一篇 →</a>
        {% endif %}
      </div>
    </div>
  {% endif %}
{% endif %}
```

- [ ] **Step 3: `_layouts/post.html` front matter 挂载**

将：
```yaml
tail_includes:
  - related-posts
  - post-nav
  - comments
```
改为：
```yaml
tail_includes:
  - series-nav
  - related-posts
  - post-nav
  - comments
```

- [ ] **Step 4: 构建并断言**

Run: `bash /d/Ruby33-x64/bin/bundle exec jekyll build`
```bash
grep -o '本系列《稳定性治理》第 5/5 篇' _site/posts/alerting-engine/index.html
grep -o 'href="/blog/posts/distributed-tracing/">← 上一篇</a>' _site/posts/alerting-engine/index.html
grep -o '本系列《后端架构》第 1/2 篇' _site/posts/reliable-message-consume/index.html
grep -o 'href="/blog/posts/batch-data-fix-file-ledger/">下一篇 →</a>' _site/posts/reliable-message-consume/index.html
grep -c 'series-nav' _site/posts/alerting-engine/index.html
```
Expected: 全部命中；末条 ≥1。

- [ ] **Step 5: 提交**

```bash
git add _includes/series-nav.html _layouts/post.html
git commit -m "系列导航：文章尾部显示系列进度与前后篇链接"
```

---

### Task 5: `docs/influence/` 产出物（4 份）

**Files:**
- Create: `docs/influence/github-profile-readme-draft.md`
- Create: `docs/influence/juejin-launch-kit.md`
- Create: `docs/influence/metrics-template.md`
- Create: `docs/influence/account-setup-guide.md`

- [ ] **Step 1: `github-profile-readme-draft.md`（可直接粘贴到同名仓库）**

必含要素：一句话定位（后端工程师 / 后端架构与稳定性治理的 0→1）、写作原则一行、两条系列线各一行、7 篇代表作链接（指向 https://cheneyzhang93.github.io/blog/posts/&lt;slug&gt;/）、博客入口与 RSS。全文中文、无 emoji。

- [ ] **Step 2: `juejin-launch-kit.md`**

必含：资料页简介（短版 ≤60 字 + 长版 ≤200 字，链接指回博客 About）；两个专栏（「稳定性治理」「后端架构」）名称与简介；封面规范（1200×630、与站点预览图同源视觉、中文标题字数上限、存放 `docs/influence/assets/`）；发布 5 步 SOP；原创策略（保守优先、保留原文链接）；互动 SOP（24h 回评、每周话题、拒绝无效互赞）。

- [ ] **Step 3: `metrics-template.md`**

月度快照模板：表头（月份）、掘金指标（发布数/阅读/收藏/评论/关注）、站点指标（周 UV、来源结构 top、掘金导流占比）、GSC（收录页数、展示词 top10）、四目标对照备注、SOP 调整项。落位规则：每月复制为 `metrics-YYYY-MM.md` 填写。

- [ ] **Step 4: `account-setup-guide.md`（用户操作单）**

逐步步骤：① giscus（仓库 Settings → Features 勾选 Discussions；giscus.app 选择 cheneyzhang93/blog、Announcements 分类、pathname 映射 → 记录 repo_id/category_id；安装 giscus App）→ 把两值回传；② GSC（添加"URL 前缀"资源 `https://cheneyzhang93.github.io/blog/` → HTML 标记取 `<meta>` 内容的 code）与 Bing 站长（同法取码）→ 回传；③ GitHub Profile 仓库（新建仓库名=用户名 → 粘贴 README 草稿）；④ 掘金（注册、资料、建两专栏，用 launch-kit 文案）。

- [ ] **Step 5: 提交**

```bash
git add docs/influence/github-profile-readme-draft.md docs/influence/juejin-launch-kit.md docs/influence/metrics-template.md docs/influence/account-setup-guide.md
git commit -m "影响力方案产出物：Profile README 草稿、掘金套件、月度快照模板与账号操作单"
```

---

### Task 6: 掘金封面模板图

**Files:**
- Create: `docs/influence/assets/juejin-cover-template.png`

- [ ] **Step 1: 参考站点预览图视觉**

Read `assets/images/preview.png`，提取视觉语言（配色/构图/元素）。

- [ ] **Step 2: 生成模板图**

用图像生成工具产出一张 1200×630（或 3:2 后裁切）封面模板：与预览图同源风格、留出标题安全区；避免图像内中文长文本（生成文字易失真），图形元素为主。
保存/移动到 `docs/influence/assets/juejin-cover-template.png`。

- [ ] **Step 3: 核验文件**

Run: `ls -la docs/influence/assets/`
Expected: 文件存在、为 PNG、尺寸 1200×630 量级。

- [ ] **Step 4: 提交**

```bash
git add docs/influence/assets/juejin-cover-template.png
git commit -m "新增掘金封面模板图（与站点预览图同源视觉）"
```

---

### Task 7: giscus 接线（Blocked：等用户提供 repo_id / category_id）

**Files:**
- Modify: `_config.yml`（comments 区块）

- [ ] **Step 1: 填入参数**（值由用户从 giscus.app 提供）

将：
```yaml
comments:
  provider: # [disqus | utterances | giscus]
```
改为（示例结构，`<>` 处替换为用户回传值）：
```yaml
comments:
  provider: giscus
  giscus:
    repo: cheneyzhang93/blog
    repo_id: <用户回传>
    category: Announcements
    category_id: <用户回传>
    mapping: pathname
    lang: zh-CN
    reactions_enabled: 1
```

- [ ] **Step 2: 构建并断言**

Run: `bash /d/Ruby33-x64/bin/bundle exec jekyll build`
```bash
grep -c 'giscus.app/client.js' _site/posts/alerting-engine/index.html
```
Expected: ≥1；本地浏览器打开文章页可见评论区（GitHub 登录可发）。

- [ ] **Step 3: 提交**

```bash
git add _config.yml
git commit -m "接入 giscus 评论（GitHub Discussions）"
```

---

### Task 8: GSC / Bing 站长验证（Blocked：等用户提供验证码）

**Files:**
- Modify: `_config.yml`（webmaster_verifications）

- [ ] **Step 1: 填码**

将：
```yaml
webmaster_verifications:
  google: # fill in your Google verification code
  bing: # fill in your Bing verification code
```
改为（值由用户回传）：
```yaml
webmaster_verifications:
  google: <用户回传的 content 值>
  bing: <用户回传的 content 值>
```

- [ ] **Step 2: 构建并断言**

Run: `bash /d/Ruby33-x64/bin/bundle exec jekyll build`
```bash
grep -o '<meta name="google-site-verification"[^>]*>' _site/index.html
grep -o '<meta name="msvalidate.01"[^>]*>' _site/index.html
```
Expected: 两条 meta 均出现且值正确。

- [ ] **Step 3: 提交并提醒用户在 GSC 后台提交 sitemap**

```bash
git add _config.yml
git commit -m "接入 Google/Bing 站长验证"
```
（用户侧：GSC → 资源 → Sitemaps → 提交 `sitemap.xml`；Bing 同步）

---

### Task 9: 全量验收（CI 同构）

- [ ] **Step 1: CI 同构构建 + html-proofer**

```bash
bash /d/Ruby33-x64/bin/bundle exec jekyll clean
JEKYLL_ENV=production bash /d/Ruby33-x64/bin/bundle exec jekyll b -d "_site/blog"
bash /d/Ruby33-x64/bin/bundle exec htmlproofer _site --disable-external --ignore-urls "/^http:\/\/127.0.0.1/,/^http:\/\/0.0.0.0/,/^http:\/\/localhost/"
```
Expected: `no issues`（与 CI pages-deploy.yml:56 同参）。

- [ ] **Step 2: 统计与 PWA 注入复核（production 产物）**

```bash
grep -c 'gc.zgo.at' _site/blog/index.html _site/blog/posts/alerting-engine/index.html
grep -c 'app.min.js' _site/blog/index.html
```
Expected: analytics 与 PWA 均注入（>0）。随后恢复默认构建供预览：`bash /d/Ruby33-x64/bin/bundle exec jekyll build`

- [ ] **Step 3: 视觉抽查（既有无头 Chrome 方法）**

亮/暗两模式截图：首页、关于页、alerting 文章页（重点：署名链接、系列导航、表格、目录）；点击链路：署名 → 关于页 → GitHub。

- [ ] **Step 4: 对照 spec §13 清单勾选**

九项中除 giscus（Task 7 未接线则标注待办）外全部通过；输出验收记录写入任务备注。

- [ ] **Step 5: 收尾**

`git status --short` 确认无本任务外的脏文件；推送与线上部署由用户确认后执行。

---

## Self-Review 记录

- **Spec 覆盖**：A→T1+T2；B→T6（og 图已由 6f557e3 落地）；C→T8；D→T7（GoatCounter 已完成 9b36c27）；E→T3+T4；F→T2；产出物与操作单→T5；验收→T9。运营节奏（W3–W8 同步/复盘）属人工运营，不在本计划编码范围。
- **占位扫描**：`<用户回传>` 仅出现在 Blocked 任务（T7/T8），为外部输入占位，已显式标注依赖；其余步骤均含完整代码/命令。
- **一致性**：series 值统一为「稳定性治理 / 后端架构」（T2/T3/T4 一致）；文章 slug 与现有 about 链接及 permalink `/posts/:title/` 一致；commit 路径均为明确文件。
