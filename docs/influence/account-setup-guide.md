# 账号操作单（需要你本人执行）

> 本文件汇总影响力方案中所有「依赖外部账号」的动作（spec §11）。每步完成后把对应值回传给我，我完成站内接线、构建验证与部署前的提交。
> 未完成不阻塞站内基建；建议 W1 内做完。

**当前进度（2026-09-17）**

- ✅ GoatCounter 已注册（站点码 `cheney`，站内接线已完成并提交）
- ✅ 掘金账号已注册、资料已完善、两个专栏已建：https://juejin.cn/user/1999384016586138
- ✅ blog 仓库 Discussions 已开启；GSC/Bing 验证码已回传并接线（待部署后回后台点「验证」）
- ✅ GitHub Profile 仓库已创建并粘贴 README：https://github.com/cheneyzhang93
- ⏳ 待办：安装 giscus app 并回传 repo_id / category_id

## 1. giscus 评论（回传两个值）

> 已核实：`cheneyzhang93/blog` 是**公开仓库、非 fork**——Discussions 一定可以开启；若在页面上找不到入口，多半是没在电脑浏览器上打开对的设置页（手机 App 不支持开 Discussions）。
> 入口路径：仓库页顶部 **Settings** → 左侧 **General** → 向下滚到 **Features** 区块 → 勾选 **Discussions**（中文界面显示为「讨论」）。勾选即生效，仓库顶部随即出现 Discussions 标签页。

1. 打开 https://github.com/cheneyzhang93/blog/settings ，在 Features 区块勾选 **Discussions**；
2. 打开 https://giscus.app/zh-CN ，「仓库」一栏填 `cheneyzhang93/blog`；若提示 giscus app 未安装，按页面提示把 App 安装到该仓库（授权时选 `cheneyzhang93/blog` 即可）；
3. 「页面 ↔️ discussion 映射关系」选 **pathname**；
4. 「Discussion 分类」选 **Announcements**（默认分类就有；下拉里没有的话选 General 也可以）；
5. 页面下方「启用 giscus」生成的代码里，把 **data-repo-id** 与 **data-category-id** 两个值复制回传。

回传格式：

```
repo_id: R_kgDOxxxxxx
category_id: DIC_kwDOxxxxxx
```

## 2. Google Search Console（回传验证码）

> 顺序很重要：**先把码回传给我 → 我填入配置并部署 → 你再点「验证」**。在验证码上线之前点验证会失败。

1. 打开 https://search.google.com/search-console ；
2. 左上角「添加资源」→ 选右侧栏的 **网址前缀**（不要选「网域」，github.io 没有 DNS 权属）；
3. 填入 `https://cheneyzhang93.github.io/blog/`（含结尾斜杠）→ 继续；
4. 验证方式选 **HTML 标记**，页面会显示一段 `<meta name="google-site-verification" content="xxxx" />`；
5. 把 content="…" 引号里的那串复制回传（不要点验证，等我部署完再点）。

## 3. Bing 站长（回传验证码）

1. 打开 https://www.bing.com/webmasters → 添加站点 → 填 `https://cheneyzhang93.github.io/blog/`；
2. 验证方式选 **Meta 标记**（或「HTML Meta tag」），复制 content 值回传；
3. 同样：回传后等我部署完，再回后台点验证。

> 两个验证码回传后我填入站点配置并部署；随后你在 GSC / Bing 后台 **Sitemaps** 提交 `https://cheneyzhang93.github.io/blog/sitemap.xml`。

## 4. GitHub Profile 仓库（约 10 分钟）

> 先厘清一个容易混淆的地址：https://cheneyzhang93.github.io/ 是你已有的**个人主页站点**（来自另一个仓库 `cheneyzhang93.github.io`），不是 GitHub 的资料页。GitHub 个人资料页地址是 **https://github.com/cheneyzhang93**，目前还没有 README；本步骤就是给它挂上内容。

1. 打开 https://github.com/new ；
2. **Repository name 填 `cheneyzhang93`**（必须与用户名一字不差；不是 blog，也不是 cheneyzhang93.github.io）；
3. 可见性选 **Public**，勾选 **Add a README file**，点 Create repository；
4. 在新仓库的 README 页点铅笔图标进入编辑，打开 `docs/influence/github-profile-readme-draft.md`，把从「后端工程师 · 后端架构与稳定性治理的 0→1」开始的分节整段粘贴进去（文档开头的用法说明不要粘）；
5. 点 Commit changes；
6. 打开 https://github.com/cheneyzhang93 即可看到个人主页展示该 README。

## 5. 掘金（注册与开栏，W3 前完成）

1. https://juejin.cn 注册；完善资料：昵称 Cheney，介绍用 `docs/influence/juejin-launch-kit.md` 的短版/长版；
2. 创建两个专栏：**稳定性治理**、**后端架构**（简介取 kit 文案）；
3. 暂不发布；W3 起按 kit 的存量同步节奏（2 篇/周、发布时间升序）开始同步，新文同日晚间随发。

## 完成后回传清单

- [ ] giscus：repo_id / category_id
- [x] GSC：已验证码已接线（aV92…GuM）
- [x] Bing：msvalidate.01 已接线（5574…9724）
- [x] GitHub Profile 仓库已创建（贴草稿）
- [x] GoatCounter 已注册（站点码 cheney，站内已接线）
- [x] 掘金账号、资料与两个专栏已创建（W3 起按 kit 节奏同步）
