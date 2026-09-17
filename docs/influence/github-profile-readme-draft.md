# GitHub Profile README 草稿

> 用法：GitHub 新建仓库 `cheneyzhang93`（仓库名与用户名同名才会显示在个人主页），勾选 Public + Add a README，把下方「后端工程师 · 后端架构与稳定性治理的 0→1」起的分节内容整段粘贴为 README.md 提交。
> 对应操作：`docs/influence/account-setup-guide.md` 第 4 节。

---

## 后端工程师 · 后端架构与稳定性治理的 0→1

记录系统的从 0 到 1：怎么设计、为什么这样取舍、实测结果如何、边界在哪里。

每篇只讲一个子系统，沿同一条线展开：设计空间 → 逐决策依据 → 机制细节 → 实测纠偏 → 边界与代价；案例来自真实工程实践、已完全脱敏。

### 稳定性治理（五部曲）

1. [从零建立测试体系：分层、基座、断言与回归纪律](https://cheneyzhang93.github.io/blog/posts/test-system-from-scratch/)——223 个用例的从零建设与三笔学费
2. [API 契约治理：OpenAPI 3 落地、契约守护与生效边界](https://cheneyzhang93.github.io/blog/posts/api-contract-governance/)——接口变更「改了就拦得住」
3. [日志治理：统一异常出口、结构化日志与字段契约](https://cheneyzhang93.github.io/blog/posts/structured-logging/)——可观测三件套之一
4. [链路追踪治理：W3C 标准落地、上下文传播与 Agent 取舍](https://cheneyzhang93.github.io/blog/posts/distributed-tracing/)——可观测三件套之二
5. [告警治理：事件引擎、通道自建与慢 SQL 感知](https://cheneyzhang93.github.io/blog/posts/alerting-engine/)——可观测三件套之三

### 后端架构（工程任务复盘）

- [收件箱模式：三方推送消息的可靠消费设计与实践](https://cheneyzhang93.github.io/blog/posts/reliable-message-consume/)——落库即应答与四态状态机
- [批量数据修复：文件账本驱动的可回滚设计](https://cheneyzhang93.github.io/blog/posts/batch-data-fix-file-ledger/)——文件账本、幂等与回滚

**博客**：[cheneyzhang93.github.io/blog](https://cheneyzhang93.github.io/blog/) · **订阅**：[RSS](https://cheneyzhang93.github.io/blog/feed.xml) · **关于**：[关于我](https://cheneyzhang93.github.io/blog/about/)
