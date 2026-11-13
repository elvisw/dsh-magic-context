# 研究与决策总索引

本目录把聊天中已确认的需求、固定版本研究和后续规划串联起来，避免只在会话中保留关键结论。以下资料均已落盘；研究完成不等于插件已经实现。

> **2026-11-13 迁移：**规划与研究资料已迁入 fork 仓库 [dsh-plugin-magic-context](https://github.com/elvisw/dsh-plugin-magic-context)；本文件中的相关链接已改为 fork 地址。下文“研究目录”等路径均指 fork 仓库内的 `_bmad-output/`。

## 后续实测与 fork 交接（2026-11-10）

- [Fork 编程智能体轻量交接](https://github.com/elvisw/dsh-plugin-magic-context/blob/main/_bmad-output/initiative-dsh-magic-context/handoff-fork-development/handoff-fork-development.md)：当前接手入口，区分已验证事实、拟议恢复契约和实施授权。
- [Windows＋真实 DSH 隔离验收](https://github.com/elvisw/dsh-plugin-magic-context/blob/main/_bmad-output/initiative-dsh-magic-context/acceptance-windows-dsh-2026-11-09/acceptance-report.md)：现成插件 `0.11.1` 的运行证据及 #13/#14 后续澄清。
- [竞品研究及后续补充](https://github.com/elvisw/dsh-plugin-magic-context/blob/main/_bmad-output/initiative-dsh-magic-context/research-context-plugin-landscape/research-context-plugin-landscape.md)：优先验证现成方案与贡献差距，而非从零重做。

下文固定版本、阶段0及“未运行应用测试/模型”等状态属于早期独立移植研究，不涵盖后续现成插件验收；原始记录保留，不自动成为 fork 的需求或实现契约。

## 需求与决定

- [用户需求与决定记录](user-input-and-decisions.md)：完整功能范围、首要痛点、已批准范围与未决事项。
- [initiative](https://github.com/elvisw/dsh-plugin-magic-context/blob/main/_bmad-output/initiative-dsh-magic-context/initiative-dsh-magic-context.md)。
- [PRD草案](https://github.com/elvisw/dsh-plugin-magic-context/blob/main/_bmad-output/initiative-dsh-magic-context/prd-dsh-magic-context/prd-dsh-magic-context.md)及其[过程日志](https://github.com/elvisw/dsh-plugin-magic-context/blob/main/_bmad-output/initiative-dsh-magic-context/prd-dsh-magic-context/.memlog.md)。
- [设计草案](../superpowers/specs/2026-11-07-dsh-magic-context-design.md)。
- [阶段0验证计划](../superpowers/plans/2026-11-07-stage0-context-lifecycle.md)：仅授权规划，尚未批准执行方式或开始实现。

## 固定版本研究及原始摘要

研究目录：`_bmad-output/initiative-dsh-magic-context/research-context-lifecycle/`。

| 资料 | 入口 |
|---|---|
| 获批研究问题、预算、证据规则与版本 | [brief.md](https://github.com/elvisw/dsh-plugin-magic-context/blob/main/_bmad-output/initiative-dsh-magic-context/research-context-lifecycle/brief.md) |
| 综合报告与源码来源 | [research-context-lifecycle.md](https://github.com/elvisw/dsh-plugin-magic-context/blob/main/_bmad-output/initiative-dsh-magic-context/research-context-lifecycle/research-context-lifecycle.md) |
| DSH定位 | [locator-dsh.md](https://github.com/elvisw/dsh-plugin-magic-context/blob/main/_bmad-output/initiative-dsh-magic-context/research-context-lifecycle/digests/locator-dsh.md) |
| MC定位 | [locator-mc.md](https://github.com/elvisw/dsh-plugin-magic-context/blob/main/_bmad-output/initiative-dsh-magic-context/research-context-lifecycle/digests/locator-mc.md) |
| DSH请求、变更与持久化 | [analyzer-q1-dsh.md](https://github.com/elvisw/dsh-plugin-magic-context/blob/main/_bmad-output/initiative-dsh-magic-context/research-context-lifecycle/digests/analyzer-q1-dsh.md) |
| MC缩减与恢复 | [analyzer-q2-mc.md](https://github.com/elvisw/dsh-plugin-magic-context/blob/main/_bmad-output/initiative-dsh-magic-context/research-context-lifecycle/digests/analyzer-q2-mc.md) |
| 共享资产与宿主边界 | [analyzer-q3-mc.md](https://github.com/elvisw/dsh-plugin-magic-context/blob/main/_bmad-output/initiative-dsh-magic-context/research-context-lifecycle/digests/analyzer-q3-mc.md) |
| 两端已有测试的具体契约 | [analyzer-q4-tests.md](https://github.com/elvisw/dsh-plugin-magic-context/blob/main/_bmad-output/initiative-dsh-magic-context/research-context-lifecycle/digests/analyzer-q4-tests.md) |
| DSH独立复核 | [verifier-dsh.md](https://github.com/elvisw/dsh-plugin-magic-context/blob/main/_bmad-output/initiative-dsh-magic-context/research-context-lifecycle/digests/verifier-dsh.md) |
| MC独立复核 | [verifier-mc.md](https://github.com/elvisw/dsh-plugin-magic-context/blob/main/_bmad-output/initiative-dsh-magic-context/research-context-lifecycle/digests/verifier-mc.md) |
| 测试证据独立复核 | [verifier-tests.md](https://github.com/elvisw/dsh-plugin-magic-context/blob/main/_bmad-output/initiative-dsh-magic-context/research-context-lifecycle/digests/verifier-tests.md) |
| 分仓库关键源码锚点 | [claims-dsh.md](https://github.com/elvisw/dsh-plugin-magic-context/blob/main/_bmad-output/initiative-dsh-magic-context/research-context-lifecycle/claims-dsh.md)、[claims-mc.md](https://github.com/elvisw/dsh-plugin-magic-context/blob/main/_bmad-output/initiative-dsh-magic-context/research-context-lifecycle/claims-mc.md) |
| 29处锚点的机械检查结果 | [verification-anchors.json](https://github.com/elvisw/dsh-plugin-magic-context/blob/main/_bmad-output/initiative-dsh-magic-context/research-context-lifecycle/verification-anchors.json) |
| 事实演变与15项语义复核日志 | [.memlog.md](https://github.com/elvisw/dsh-plugin-magic-context/blob/main/_bmad-output/initiative-dsh-magic-context/research-context-lifecycle/.memlog.md) |
| 文档结构、措辞及设计可执行性检查 | [structure](https://github.com/elvisw/dsh-plugin-magic-context/blob/main/_bmad-output/initiative-dsh-magic-context/research-context-lifecycle/review-structure.md)、[prose](https://github.com/elvisw/dsh-plugin-magic-context/blob/main/_bmad-output/initiative-dsh-magic-context/research-context-lifecycle/review-prose.md)、[builder](https://github.com/elvisw/dsh-plugin-magic-context/blob/main/_bmad-output/initiative-dsh-magic-context/research-context-lifecycle/review-builder.md) |

独立分析稿中的“awaiting verification”和早期日志中的unverified是当时的历史状态，不改写原始过程记录；最终状态以综合报告和后续verifier记录为准。

## 核心结论与限制

规划阶段另核查了原生pruner、flush、pre-step、CLI隔离、SDK解析和许可，见[补充源码核查](planning-followup.md)。这些补充不混入原四问题的预算与复核计数。

1. MC通过稳定身份、排队缩减、持久化决定与确定性重放移除可见内容，ctx_expand从原始来源产生新的工具结果；不需要重新执行原工具。
2. DSH请求从持久日志派生并冻结，agent/request和llm/stream不是任意正文变更入口。原生surface变更是待验证的适配方向，不是已实现事实。
3. 上游可复用的是内部源码、提示词与测试资产，没有已发布的通用core SDK可直接替换宿主。
4. 缓存稳定是控制前缀改写，不是永久命中保证；已有测试也不等于真实提供商费用/命中率实测。
5. 文本/工具输入输出恢复与二进制附件还原不同；Windows部分恢复、跨存储部分提交、插件禁用后体验仍需验证或决策。

未运行应用测试、未安装产品依赖、未调用模型。临时clone不属于仓库；固定提交及逐项路径足以重新定位源码。不要依赖检查JSON中的本机绝对路径来运行产品。

## 可复核来源

- [2026-11-07上游基线检查](upstream-baseline-check-2026-11-07.md)
- [2026-11-07来源核对收尾](source-check-handoff-2026-11-07.md)：latest release与现基线相同，默认分支只追加CLI发布README复制步骤；本次不移动基线。

- [MC固定快照](https://github.com/cortexkit/magic-context/tree/33fb6ac9a39c8bb299776b6b514ea0815a15a225)：v0.49.40。
- [DSH固定快照](https://github.com/deepseek-ai/deepseek-harness/tree/fcc0ad462172522aaf5c93a7b2e01f6b9f0b0159)。

可在独立参考目录clone对应仓库后checkout上述SHA；不要在本项目中检出上游覆盖研究文件。早期聊天使用过MC另一个master提交，其观察不替代本次固定release证据。阶段0计划中的接口和阈值是实验提议，不混入源码事实结论。
