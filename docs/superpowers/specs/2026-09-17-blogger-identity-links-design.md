# 博客身份链接体系设计（方案 B）

日期：2026-09-17
状态：已确认
环境：Jekyll 4.3.3 · jekyll-theme-chirpy 7.0.1（Gemfile 锁定）· jekyll-seo-tag 2.8.0

## 1. 背景与问题

站点是 GitHub Pages 项目站（url `https://cheneyzhang93.github.io`，baseurl `/blog`），存在四处「读者无法从内容链接到博主本人」的断点：

1. 文章署名（byline）与页脚作者名是纯文本，不可点击——`site.social.links` 为空（两条候选均被注释）；
2. BlogPosting 结构化数据无 `author` 字段，无 `<meta name="author">`——`_config.yml` 无 `author` 键；
3. 站内「关于」页此前为占位符（现有未提交草稿已补全，但缺 GitHub 入口）；
4. 机读身份缺失：无 `rel="me"` 声明、无站级 Person 实体，搜索引擎无法把内容归并到同一人名下。

读者渠道决策（已澄清）：只纳入既有渠道（GitHub、邮箱、RSS），不新增外部账号；邮箱保持现状不改。

## 2. 目标与非目标

### 目标

- **人读链路**：署名 / 页脚 `Cheney` → 关于页（身份中枢）→ GitHub / 邮箱 / 友链，全链路可点击；
- **机读链路**：文章 `BlogPosting.author(Person, url→关于页)` → 关于页 `ProfilePage + sameAs(GitHub)` → 外部身份；全站 `rel="me"` + 非文章页站级 `Person` 实体，指向同一实体。

### 非目标

- 不新增外部渠道、不改邮箱、不加文末作者卡、不改 feed、不动 `_posts/*`。

## 3. 现状核验结论（事实基础，均为源码级验证）

| # | 结论 | 验证方式 |
|---|------|----------|
| 1 | **feed 无需改动**：Feed 由 Chirpy 主题自带 `assets/feed.xml` 提供（`permalink: /feed.xml`），作者名取 `site.social.name`、主页取 `{{ "/" \| absolute_url }}`，与 `author:` 配置无耦合；jekyll-feed 不在 Gemfile.lock（未激活） | 主题源码 + 构建产物 |
| 2 | **新增 `author:` 是纯增量**：全主题 grep 确认 `site.author` 仅被 jekyll-seo-tag 消费（`author_drop.rb` 解析顺序：`page.author` → `page.authors[0]` → `site.author`），Chirpy 自身不读 | 主题 gem 全量 grep |
| 3 | **`homepage_or_about?` 只匹配首页与关于页**：`HOMEPAGE_OR_ABOUT_REGEX = %r!^/(about/)?(index.html?)?$!` 作用于 `page.url`（不含 baseurl），`/` 与 `/about/` 命中；二者才输出 `sameAs`（取 `site.social.links`），且 `page.seo.links` 页面级值优先 | `jekyll-seo-tag-2.8.0/lib/jekyll-seo-tag/drop.rb:12,138-146,192-194` |
| 4 | **twitter 空键有副作用**：现状已输出 `<meta name="twitter:site" content="@" />`（`{% if site.twitter %}` 对非 nil Hash 为真）；新增 `author` 后 `AuthorDrop#twitter` 回退 name，会再输出 `twitter:creator @Cheney`。主题与站点均无其他 `site.twitter` 消费方 | `template.html:74-78`、`author_drop.rb:35-40`、主题+repo grep |
| 5 | **扩展点可用**：主题 `head.html:106` 以 `{% include metadata-hook.html %}` 引入站点级扩展点（主题内为占位注释），站点侧新建同名文件即以覆盖机制生效 | 主题 `head.html` |
| 6 | 文章均未设置 `page.author`，`site.author` 回退对全部文章生效 | `_posts/*` grep |

## 4. 改动方案

### 4.1 `_config.yml`

新增顶层 `author`，并让 `social.links[0]` 指向关于页（驱动 Chirpy 署名/页脚链接：主题 `footer.html` 与 `_layouts/post.html` 均取 `site.social.links[0]`）：

```yaml
author:
  name: Cheney
  url: "https://cheneyzhang93.github.io/blog/about/"

social:
  name: Cheney
  email: 13155192291@163.com        # 保持现状
  links:
    - https://cheneyzhang93.github.io/blog/about/
```

（`links` 下现有的两条注释候选行直接替换为上述 About 行；GitHub 入口由侧边栏 `_data/contact.yml` 承担，不重复进 `social.links`。）

twitter 空键注释掉（消除核验结论 #4 的两处无效 meta；恢复路径同步保留注释说明）：

```yaml
# twitter 未启用（无账号）：恢复时填 username 并取消 _data/contact.yml 中图标注释
# twitter:
#   username:
```

**新增后由 seo-tag 自动获得的输出**：`<meta name="author">`（全站）与 BlogPosting JSON-LD `author: {"@type":"Person","name":"Cheney","url":".../about/"}`（`json_ld_drop.rb:38-53`）。

### 4.2 `_tabs/about.md`（增量，不重写现有草稿）

front matter 增加：

```yaml
seo:
  type: ProfilePage
  links:
    - https://github.com/cheneyzhang93
```

正文「联系与订阅」小节增加一行：

```markdown
- **GitHub**：[cheneyzhang93](https://github.com/cheneyzhang93)——代码与实践的补充；
```

### 4.3 `index.html`

front matter 增加（修正首页 `sameAs` 语义：应指向外部身份 GitHub，而非站内关于页）：

```yaml
seo:
  links:
    - https://github.com/cheneyzhang93
```

### 4.4 新建 `_includes/metadata-hook.html`

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

## 5. 关键决策与依据

| 决策 | 依据 |
|---|---|
| 署名/页脚指向**关于页**而非 GitHub | 署名惯例指向站内 bio 页（作者身份中枢）；GitHub 属外部身份，走机读链路（rel=me / sameAs），两种意图不混淆 |
| 首页/关于页用**页面级 `seo.links`** | 见核验结论 #3：否则首页与关于页会把「站内关于页 URL」当 sameAs 输出（自指，语义错误）；页面级 `seo.links` 覆盖后指向 GitHub，`social.links` 的 SEO 角色被完全遮蔽，仅保留「署名链接目标」单一职责 |
| `Person` JSON-LD **仅非文章页** | 文章页已有 seo-tag 输出的 `BlogPosting.author` Person；再输出独立 Person 会形成无 `@id` 关联的重复实体。seo-tag 不支持给 author 注入自定义 `@id`（`json_ld_drop.rb:38-53`），故以「url 一致性」间接归并，不做双实体 |
| `author` 不写 `type` | JSON-LD 默认 `Person`；`VALID_AUTHOR_TYPES` 仅允许 Person/Organization（`json_ld_drop.rb:23`），写 `type` 无收益 |
| 关于页 `type: ProfilePage` | schema.org 标准类型；现状该页命中 `homepage_or_about?` 继承 `WebSite` 类型，语义不准 |
| Person 的 name 取 `site.author.name` | 与 seo-tag 输出的 author 同源，避免两处名称漂移 |
| 关于页/首页 front matter 中 GitHub URL 为字面量 | front matter 不经 Liquid 渲染，无法引用变量；维护点见第 7 节 |

## 6. 验证清单（构建后逐项核验）

1. 署名/页脚为链接：文章页 HTML 含 `<a href="https://cheneyzhang93.github.io/blog/about/">Cheney</a>`；
2. 全站含 `<meta name="author" content="Cheney">`；
3. 文章页 BlogPosting JSON-LD 含 `"author":{"@type":"Person","name":"Cheney","url":"https://cheneyzhang93.github.io/blog/about/"}`；
4. 首页 JSON-LD `@type` 保持 `WebSite`，`sameAs` = `["https://github.com/cheneyzhang93"]`；
5. 关于页 JSON-LD `@type` = `ProfilePage`，`sameAs` = GitHub；
6. `<link rel="me" href="https://github.com/cheneyzhang93">` 全站存在；独立 Person JSON-LD 仅出现在非文章页（文章页无该块）；
7. `<meta name="twitter:site">` 与 `twitter:creator` 无效标签消失，`twitter:card`/`twitter:title` 保留；
8. `feed.xml` 除 `<updated>` 构建时间戳外与改动前无差异（作者名、入口链接、条目内容不变，订阅端零影响）；
9. 本地预览（端口 4123）点击链路：署名 → 关于页 → GitHub。

## 7. 风险与边界

- **主题升级**：`metadata-hook.html` 依赖主题 `head.html:106` 的 include 位置；Gemfile 已锁 7.0.1、seo-tag 锁 2.8.0，升级时需复验该点与 `HOMEPAGE_OR_ABOUT_REGEX` 行为；
- **GitHub URL 维护点**：字面量出现于 `metadata-hook.html`（1 处，文件内已变量化）、`about.md` front matter、`index.html` front matter；更换账号需同步 3 个文件；
- 关于页正文引用的 `/friends/` 页为并行在建内容（`_tabs/friends.md`、`_data/friends.yml` 未提交），本设计不改动它。

## 8. 参考

- Google Search Central — Article structured data（author 为含 `url` 的 Person）：https://developers.google.cn/search/docs/data-types/article
- IndieWeb — rel-me（身份验证标准）：https://indieweb.org/rel-me
- Building the IndieWeb: A Technical Guide for Developers：https://dev.to/rosgluk/building-the-indieweb-a-technical-guide-for-developers-4f79
- 本地源码：`jekyll-seo-tag-2.8.0/lib/jekyll-seo-tag/{drop,author_drop,json_ld_drop}.rb`、`template.html`；`jekyll-theme-chirpy-7.0.1/_includes/{head,footer}.html`、`assets/feed.xml`
