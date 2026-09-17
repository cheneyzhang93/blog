# 账号操作单（需要你本人执行）

> 本文件汇总影响力方案中所有「依赖外部账号」的动作（spec §11）。每步完成后把对应值回传给我，我完成站内接线、构建验证与部署前的提交。
> 未完成不阻塞站内基建；建议 W1 内做完。

## 1. giscus 评论（回传两个值）

1. 打开博客仓库 `cheneyzhang93/blog` → **Settings** → General → **Features** → 勾选 **Discussions**；
2. 打开 https://giscus.app ，Repository 填 `cheneyzhang93/blog`（首次会提示安装 giscus App，按提示授权到该仓库）；
3. Discussion 分类选 **Announcements**；页面 ↔ Discussion 映射选 **pathname**；
4. 页面下方生成的配置中，把 **repo_id** 与 **category_id** 两个值复制回传。

回传格式：

```
repo_id: R_kgDOxxxxxx
category_id: DIC_kwDOxxxxxx
```

## 2. Google Search Console（回传验证码）

1. https://search.google.com/search-console → 添加资源 → 选 **网址前缀** → 填 `https://cheneyzhang93.github.io/blog/`；
2. 验证方式选 **HTML 标记**，复制 `<meta name="google-site-verification" content="xxxx" />` 里 **content 的值**回传。

## 3. Bing 站长（回传验证码）

1. https://www.bing.com/webmasters → 添加站点 → 填 `https://cheneyzhang93.github.io/blog/`；
2. 选 meta 验证方式，复制 content 值回传。

> 两个验证码回传后我填入站点配置并部署；随后你在 GSC / Bing 后台 **Sitemaps** 提交 `https://cheneyzhang93.github.io/blog/sitemap.xml`（W2 末完成即可）。

## 4. GitHub Profile 仓库（约 10 分钟）

1. GitHub 新建仓库，仓库名填 **`cheneyzhang93`**（与用户名同名才会显示在个人主页）；
2. 勾选 Public + Add a README，创建；
3. 打开 `docs/influence/github-profile-readme-draft.md`，把「后端工程师 · 后端架构与稳定性治理的 0→1」起的分节整段粘贴为 README.md，提交。

## 5. 掘金（注册与开栏，W3 前完成）

1. https://juejin.cn 注册；完善资料：昵称 Cheney，介绍用 `docs/influence/juejin-launch-kit.md` 的短版/长版；
2. 创建两个专栏：**稳定性治理**、**后端架构**（简介取 kit 文案）；
3. 暂不发布；W3 起按 kit 的存量同步节奏（2 篇/周、发布时间升序）开始同步，新文同日晚间随发。

## 完成后回传清单

- [ ] giscus：repo_id / category_id
- [ ] GSC：google-site-verification 的 content 值
- [ ] Bing：msvalidate.01 的 content 值
- [ ] GitHub Profile 仓库已创建（贴草稿）
- [ ] 掘金账号与两个专栏已创建
