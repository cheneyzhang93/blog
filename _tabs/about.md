---
# the default layout is 'page'
title: 关于
icon: fas fa-info-circle
order: 4
---

我是 Cheney，后端工程师。这里记录系统的从 0 到 1：怎么设计、为什么这样取舍、实测结果如何、边界在哪里。

## 这个博客写什么

- **后端架构**：消息链路的可靠消费、批量数据修复这类工程任务的设计复盘；
- **稳定性治理**：测试体系、API 契约与可观测性（日志 / 链路 / 告警）的从零建设。

## 怎么写

每篇只讲一个子系统，沿同一条线展开：**设计空间 → 逐决策依据 → 机制细节 → 实测纠偏 → 边界与代价**。

案例来自真实工程实践、已完全脱敏；效果数字与实现保持一致，没做到的部分也会如实标注。

## 从这里读起

- [《收件箱模式：三方推送消息的可靠消费设计与实践》]({{ '/posts/reliable-message-consume/' | relative_url }})——落库即应答与四态状态机；
- [《从零建立测试体系：分层、基座、断言与回归纪律》]({{ '/posts/test-system-from-scratch/' | relative_url }})——223 个用例的从零建设与三笔学费；
- [《日志治理：统一异常出口、结构化日志与字段契约》]({{ '/posts/structured-logging/' | relative_url }})——可观测三件套的第一篇（另有[《链路追踪治理》]({{ '/posts/distributed-tracing/' | relative_url }})与[《告警治理》]({{ '/posts/alerting-engine/' | relative_url }})）；
- [《API 契约治理：OpenAPI 3 落地、契约守护与生效边界》]({{ '/posts/api-contract-governance/' | relative_url }})——接口变更「改了就拦得住」。

## 联系与订阅

- **邮箱**：[cheney@example.com](mailto:cheney@example.com)——交流、指正，欢迎来[友情链接]({{ '/friends/' | relative_url }})页交换友链；
- **RSS**：订阅 [feed.xml]({{ '/feed.xml' | relative_url }})，更新不遗漏。
