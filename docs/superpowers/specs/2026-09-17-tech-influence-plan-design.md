# 技术与行业影响力方案设计（以博客为核心资产）

日期：2026-09-17
状态：设计稿（已自审，待审阅）
环境：Jekyll 4.3.3 · jekyll-theme-chirpy 7.0.1（Gemfile 锁定）· GitHub Pages 项目站（url `https://cheneyzhang93.github.io`，baseurl `/blog`）
关联文档：`docs/superpowers/specs/2026-09-17-blogger-identity-links-design.md`（身份链接体系，已确认；本方案 A 项=实施其剩余部分，不重复其论证）

## 1. 背景与问题

- **内容侧**：7 篇深度文（2026-08-06 → 09-17，每周四），已天然形成两条系列线，写作结构统一（设计空间 → 决策 → 实测 → 边界）；但触达仅靠自然流量与 RSS，无任何分发渠道。
- **基建侧**：身份链接体系的「人读/机读」收口项尚未落地（署名不可点击、作者实体无 url、无 rel=me）；分析、评论、站长验证仍为空白；og:image、feed 收录数、SEO 站点身份已由并行会话落地（见 §3）。
- **结果**：内容资产触达面窄，四类影响力目标（职业机会、技术品牌、读者辐射、变现地基）缺少可积累的基建。

## 2. 目标与非目标

### 目标（四层，已确认）

1. 职业机会与行业背书；2. 技术品牌与同行认可；3. 读者辐射与社区回馈；4. 长期变现地基。

### 路线（已确认）

**路径 A「基建先行 + 系列深耕」**：对已落地基建做增量收口 + 补齐度量/评论/分发空白（约 2 周）+ 掘金单渠道分发 + 原有周更内容纪律不变；旗帜文/开源配套作为季度增强留位。

其余候选路径与排除原因：

| 路径 | 排除原因 |
|---|---|
| B 掘金冲量优先 | 核心资产（源站）停滞、受众沉淀在平台、与「以博客为核心资产」前提矛盾 |
| C 旗帜作品+行业连接 单独走 | 部分动作依赖外部反馈、节奏不可控；降级为本方案的季度增强项 |

### 非目标

- 不新增掘金以外的分发渠道（公众号/知乎/X 本期均不做）；
- 不做旗帜文与开源配套的承诺（留位，另行立项）；
- 不改写作节奏与文章结构；不改内容脱敏纪律；
- 不改邮箱对外策略（关于页维持占位邮箱现状）；
- 不改已落地的 og:image 与 feed 覆盖方案。

## 3. 现状核验结论（事实基础，均为源码级验证）

### 3.1 并行会话已落地（5898faa / 6f557e3，计入现状，本方案不重复实施）

| # | 已落地项 | 验证方式 |
|---|----------|----------|
| D1 | SEO 站点身份：`title/tagline/description` 重写；补 `author: Cheney`（当前为**字符串**形式，L26） | `git show 6f557e3` diff |
| D2 | 社交预览：`social_preview_image: assets/images/preview.png`（1200×630，L104），og:image + `twitter:card` 大图 | 同上 |
| D3 | 空的 `twitter:` 配置已移除（消除 `twitter:site="@"` 无效输出） | 同上 |
| D4 | feed 收录上限 5 → 20（站点侧覆写 `assets/feed.xml`）——RSS 归档限制已解决 | `assets/feed.xml:27` |
| D5 | 站点侧 SCSS 覆盖（表格/目录/中文字体栈）、归档/分类/标签页 title、标签规范化 | `assets/css/jekyll-theme-chirpy.scss` 等 |
| D6 | 友链页与关于页已上线；工作区干净（仅本 spec 未跟踪）——此前的并行冲突依赖已消除 | `git status`（2026-09-17） |

### 3.2 身份设计剩余缺口（本方案 A 项实施对象）

| # | 结论 | 验证方式 |
|---|------|----------|
| 1 | `author` 为字符串形式，无 `url` → 文章 JSON-LD 的 `BlogPosting.author` 缺指向关于页的 url | `_config.yml:26` |
| 2 | `social.links` 候选仍全部注释（L40-46）→ 署名/页脚为纯文本，不可点击 | `_config.yml` + `_layouts/post.html` 署名逻辑（取 `site.social.links[0]`） |
| 3 | 站点侧 `_includes/` 目录不存在 → `rel="me"` 与非文章页站级 `Person` 实体未落地 | Glob 核验 |
| 4 | `index.html` 无 seo front matter → 首页 `sameAs` 缺外部身份指向（且 `social.links` 空时 `homepage_or_about?` 无输出） | `index.html` |
| 5 | `_tabs/about.md` 无 seo front matter（ProfilePage/GitHub），正文无 GitHub 入口 | 源码核验 |

### 3.3 其余现状

| # | 结论 | 验证方式 |
|---|------|----------|
| 6 | 分析/评论/站长验证全空：`analytics`（L61-73）、`comments`（L109-129）、`webmaster_verifications`（L49-55）均未配置 | `_config.yml` |
| 7 | 主题扩展点可用：`head.html:106` 引入 `metadata-hook`（主题内为空占位）；`tail_includes` 由主题 default 布局消费，可在既有 post.html 覆盖内扩展 | 主题 gem 源码 |
| 8 | 分类天然成线：`后端架构`×2（消息消费、数据修复）、`稳定性治理`×5（测试、契约、日志、链路、告警） | `_posts/*` front matter |
| 9 | 构建链：`pages-deploy.yml` 单工作流部署；本地构建可预览（惯例端口 4123），测试组含 html-proofer 5.0.9 可作质量门 | Gemfile.lock / .github/workflows / 既有流程 |

## 4. 方案总览

### 资产结构（三层）

| 层 | 资产 | 角色 |
|---|---|---|
| 权威源 | 博客源站（7 篇深度文 + 周更节奏） | 唯一首发地、机读身份锚点 |
| 可迁移资产 | 内容系列、身份链路（人读+机读）、受众（RSS/友链/GitHub） | 不受平台算法影响的底盘 |
| 放大器 | 掘金（唯一新增主渠道） | 触达技术圈，导流回源站 |

### 边界面

1. **源站首发原则不变**：所有内容先在源站发布，掘金只做同步分发；
2. **单渠道**：掘金外不新增任何平台；
3. **受众可迁移**：不以平台粉丝为唯一目标，站点分析须能观测掘金导流占比；
4. **脱敏与如实纪律不变**：新渠道沿用既有内容纪律。

### 实施范围界定

本 spec 覆盖：身份剩余缺口收口（A）、掘金封面模板（B）、站长验证（C）、度量与评论（D）、系列化（E）、关于页作品化（F）、草稿类产出（README/简介/检查单/快照模板）、SOP 机制定义。W3 起的存量同步与月度复盘为运营执行，按 §7/§9 运转，不在实施计划内。

## 5. 站内基建（W1–W2）

### A. 身份链接体系收口（实施已确认设计的剩余部分）

按 `2026-09-17-blogger-identity-links-design.md` §4 未落地项逐项实施：

- `_config.yml`：`author` 由字符串升级为映射（`name: Cheney` + `url: https://cheneyzhang93.github.io/blog/about/`），构建后核验 `<meta name="author">` 与 `BlogPosting.author.url`；`social.links` 替换为 About 链接（驱动署名/页脚点击）；
- `index.html`：补 seo front matter（`seo.links: [GitHub]`，修正首页 sameAs 语义）；
- `_tabs/about.md`：补 seo front matter（`ProfilePage` + GitHub）与正文 GitHub 入口行；
- 新建 `_includes/metadata-hook.html`（`rel="me"` 全站 + 非文章页站级 Person JSON-LD）；
- 验证按其 §6 九项清单执行（其中 twitter 相关项已由 D3 达成，核验时确认无回归即可）。

**实施纪律**：并行会话曾同向改动此区域，实施前先复核最新 HEAD 是否已落地同项；每项按明确路径提交，避免重复与冲突。

### B. 掘金封面模板

- og 默认图已落地（D2），本项仅剩分发侧：按同源视觉规范生成掘金封面模板，按篇产出、同步时上传。

### C. 搜索引擎接入

- `webmaster_verifications.google` / `bing` 填入验证码（用户取码，实施时填入）；
- GSC 资源使用 URL 前缀 `https://cheneyzhang93.github.io/blog/`；Bing 站长同步；
- 提交 `sitemap.xml`（已生成，仅需提交与核验）；核验收录页数与展示词；
- 百度站长：github.io 收录弱，标为可选，不列为默认目标。

### D. 度量与评论

- **分析**：GoatCounter（无 cookie、个人免费额度、Chirpy 原生支持）→ `analytics.goatcounter.id`；并启用 `pageviews.provider: goatcounter`（文章页显示浏览量）；
- **回退**：若 GoatCounter 可达性/稳定性实测不佳，切 Umami（自托管）或 GA4；
- **评论**：giscus（GitHub Discussions，延续 GitHub 身份）→ `comments.provider: giscus` + repo / repo_id / category / category_id / mapping: pathname / lang: zh-CN；
- 核验：全站无 cookie、无同意横幅。

### E. 内容系列化

- `_posts/*` front matter 增量 `series` 字段（稳定性治理×5、后端架构×2），正文与时间线不动；
- 新建 `_includes/series-nav.html`：渲染「本系列第 n/共 m 篇 + 上一篇/下一篇 + 系列入口」；
- `_layouts/post.html` 的 `tail_includes` 首位加入 `series-nav`；
- About 页「从这里读起」升级为系列导览（两条线入口指向对应分类页，URL 以构建产物核验后固定）。

### F. 关于页作品化

- 在现有定位句基础上补「代表作 + 工程主张」结构（代表作即两条系列线）；
- GitHub 入口与 seo 由 A 项带入；邮箱维持现状。

## 6. 身份锚点（分发与背书侧）

| 锚点 | 内容 | 产出方式 |
|---|---|---|
| 站内 About | A/F 项 | 我实施 |
| GitHub Profile README | 定位一句话 + 代表作链接 + 联系入口 | 我产出草稿 `docs/influence/github-profile-readme-draft.md`，用户在同名仓库启用 |
| 掘金资料页 | 规范化简介，链接指回 About | 我产出文案，用户填入 |

## 7. 掘金分发 SOP

### 发布链路（每篇固定 5 步）

1. 源站周四首发（节奏不变）；
2. 当天傍晚掘金同步（粘贴 Markdown 导入；mermaid、代码块、表格逐项核验渲染）；
3. 文末附「原文首发」链接指回源站；
4. 选择对应专栏与分类/标签（与站内 series 对齐）；
5. 上传按模板生成的封面。

### 原创策略（保守优先）

优先尝试原创声明；若被判定疑似重复则改选「转载」并注明原文——两种结果下「原文链接」均保留，影响力无损。规则以掘金官方帮助中心为准，每次同步前对照最新版。

### 存量同步

W3–W6 期间 2 篇/周，按发布时间升序把 7 篇存量搬完（建立作者主页与专栏基线）；新文随后即发即同步。

### 专栏

建「稳定性治理」「后端架构」两个专栏，与站内系列一一对应。

### 互动 SOP

- 发布后 24h 内回复评论；
- 每周固定时段回访同领域作者、参与 1–2 个话题；
- 不做无意义的互赞互关。

## 8. 内容体系

- **两条线定位**：稳定性治理五部曲（测试 → 契约 → 日志 → 链路 → 告警，叙事闭环）；后端架构复盘（消息消费、数据修复）；
- **结构纪律不变**：设计空间 → 逐决策依据 → 机制细节 → 实测纠偏 → 边界与代价；
- **后续选题矩阵**（延伸位，不承诺时间）：性能与容量治理、发布与回滚、故障复盘文化；分布式一致性、缓存与异步化；
- **季度增强留位**：旗帜文（如可观测三篇升级为权威指南）、开源配套仓库（状态机/契约守护示例）。

## 9. 度量与复盘

### 三层指标

| 层 | 指标 | 节奏 |
|---|---|---|
| 基建完成度 | §13 验证清单全绿 | W2 内 100% |
| 月度表现 | 掘金：发布数、阅读、收藏、评论、关注；站点：周 UV、来源结构（掘金导流占比）；GSC：收录页数、展示词 | 每月快照 |
| 季度成果 | 搜索词进入、互链/友链数、私信与合作邀约、旗帜文/开源落地数 | 每季度对照四目标 |

### 月度快照

记录于 `docs/influence/metrics-YYYY-MM.md`（git 版本化），模板要素：掘金数据、站点访问与来源结构、GSC 收录与词、与四目标的对照备注、SOP 调整项。

### 发布检查单

每篇发布时执行：站内发布（构建 + 渲染验证）→ 掘金 5 步 → 数据登记。融入既有发布流程，不另建系统。

## 10. 里程碑与依赖

| 周 | 动作 | 完成标志 | 依赖 |
|---|---|---|---|
| W1–W2 | 站内基建 A–F 落地；用户账号侧动作并行进行 | §13 清单全绿 | 用户取验证码（C/D 项填入用） |
| W2 末 | GSC/Bing 提交 sitemap、发起收录 | 收录页数开始增长 | 同上 |
| W3–W6 | 存量同步 2 篇/周 + 新文同步；专栏建成 | 7 篇存量全上线、两专栏成型 | 掘金账号（用户） |
| W7–W8 | 首期复盘（W4/W8 快照 + 对照） | 复盘文档 + SOP 调整项 | 前三周数据 |

## 11. 用户侧操作清单

1. **GoatCounter**：注册 → 取得站点码（填入由我完成）；
2. **GitHub**：仓库开启 Discussions；在 giscus.app 生成 repo_id / category_id；安装 giscus app；
3. **GSC**：添加 URL 前缀资源 `https://cheneyzhang93.github.io/blog/` → 取 HTML 标记验证码；Bing 站长同理取码；
4. **GitHub Profile**：创建与用户名同名仓库，启用 README 草稿；
5. **掘金**：完成账号资料（用产出简介）、创建两个专栏；此后按 §7 SOP 同步。

## 12. 风险与边界

- **并行会话同向改动**：此前该区域有并行提交（D1–D6）——实施前复核最新 HEAD，逐项 diff 后落地，按明确路径提交；
- **平台规则/算法变化**：源站权威不可逆、掘金只是放大器，任何调整不伤底盘；每次同步对照官方规则；
- **账号动作依赖用户**：W1 内完成即可（操作单 §11）；未完成不阻塞站内基建；
- **跨平台渲染差异**：mermaid/外链图首次同步逐一核验，有差异再调整图片策略；
- **分析工具可达性**：GoatCounter 国内可达性需实测，预留 Umami/GA4 回退路径；
- **度量纪律**：不追虚荣指标（只看来源结构、收录、互动质量）；脱敏纪律不变。

## 13. 验证清单（构建后逐项核验）

1. 身份清单：`2026-09-17-blogger-identity-links-design.md` §6 九项在最新基线（含 D1–D6）上全部通过；
2. og:image 全站输出保持（D2 无回归），`twitter:card=summary_large_image`；
3. 文章页 giscus 评论正常加载（GitHub 登录可发评论）；
4. GoatCounter 脚本加载、无 cookie；文章页浏览量显示正常；
5. 系列导航渲染正确：系列内第 n/共 m 篇、上一篇/下一篇链接准确；
6. About 系列导览链接可达（分类页 URL 与构建产物一致）；
7. `sitemap.xml`、`robots.txt` 无意外变化；`feed.xml` 保持 20 条上限（D4 无回归）；
8. html-proofer 通过（无死链）；
9. 全站无 cookie 同意横幅；PWA 正常。

## 14. 关键决策记录

| 决策 | 依据 |
|---|---|
| 路径 A（基建先行）而非 B（冲量） | 核心资产=博客；受众须可迁移；B 的流量沉淀在平台，与前提矛盾 |
| 掘金单渠道 | 用户选定；技术圈流量与深度文适配度最高；公众号/知乎/X 留作后续可选扩展 |
| 身份收口按已确认设计执行（author 映射化、署名链接、metadata-hook） | 复用已核验的设计（seo-tag drop 行为、扩展点），不重复论证 |
| GoatCounter 而非 GA4 | 无 cookie 免同意横幅、个人免费、Chirpy 原生支持并可回显浏览量；GA4 留作回退 |
| giscus 而非 disqus/utterances | 延续 GitHub-only 身份；主题原生支持；utterances 已有替代态势 |
| 系列化用 category+series-nav 而非新页面 | 分类页已自动生成（jekyll-archives）；仅需 front matter 增量 + 一个轻量 include，最小侵入 |
| 掘金封面按模板生成，不逐篇定制 | 复用已落地的预览图视觉规范，单篇成本趋零 |

## 15. 参考

- 本仓库：`docs/superpowers/specs/2026-09-17-blogger-identity-links-design.md`（身份链接体系）
- Google Search Central — 站点验证与 sitemap：https://developers.google.cn/search/docs
- GoatCounter 文档：https://www.goatcounter.com/help
- giscus：https://giscus.app/zh-CN
- 掘金帮助中心（原创与发布规则以官方为准）：https://juejin.cn/help
