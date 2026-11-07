# dsh-magic-context

将 [Magic Context](https://github.com/cortexkit/magic-context) 的上下文管理能力移植到 [DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness)。

首要目标是让旧工具调用内容逐步退出模型上下文，同时保留稳定引用，在需要时恢复当时的原文，而不是重新运行工具或用摘要冒充原始结果。

> 当前处于研究与规划阶段，没有可安装的插件。源码研究完成不代表运行兼容性已经验证。

## 阅读入口

- [研究与决策总索引](docs/research/README.md)：已落盘的资料、来源、结论及未决事项。
- [固定版本源码研究](./_bmad-output/initiative-dsh-magic-context/research-context-lifecycle/research-context-lifecycle.md)：请求链路、可逆卸载、宿主边界和测试证据。
- [PRD 草案](./_bmad-output/initiative-dsh-magic-context/prd-dsh-magic-context/prd-dsh-magic-context.md)：保留全部原始需求，完整 V1 尚未冻结。
- [设计草案](docs/superpowers/specs/2026-11-07-dsh-magic-context-design.md)：复用/适配清单和技术风险。
- [阶段0验证计划](docs/superpowers/plans/2026-11-07-stage0-context-lifecycle.md)：六项离线验证任务，待审核与实施授权。
- [项目代理规则](AGENTS.md)：包括开发和评审子代理的成本规则。

## 已确定的方向

采用固定上游版本、复用选定源码/提示词/测试、独立维护 DSH adapter 的路线，不从零重新发明上下文缩减机制。

| 来源 | 固定基线 |
|---|---|
| Magic Context | `v0.49.40` / `33fb6ac9a39c8bb299776b6b514ea0815a15a225` |
| DSH 源码 | `fcc0ad462172522aaf5c93a7b2e01f6b9f0b0159` |
| 本机 DSH 观察版本 | `0.2.2-rc.1`；发布包与源码逐文件一致性未验证 |

阶段0范围已确认：隔离 Windows/DSH 环境、文本/JSON 工具内容、离线模型驱动。计划审核、执行方式和实现启动仍待确认；完整 V1、最终存储格式、跨 harness 数据互读与正式禁用行为尚未定案。项目面向开源发布。

## 仓库内容与边界

- `_bmad-output/`：研究简报、定位/分析/复核摘要、claims、锚点检查记录、文档检查、需求及决策日志，包含隐藏的 `.memlog.md`。
- `docs/`：研究导航、设计草案和阶段0计划。
- `_bmad/config.toml`：可移植的项目配置。BMad 安装脚本及本地用户覆盖不纳入版本管理；需要运行工作流时在本机安装 BMad 并选择 `initiative-dsh-magic-context`。
- 临时参考 clone 不上传；报告保存远程仓库、固定提交、相对源码路径和行号，可重新获取证据。历史检查文件中的本机路径仅记录当时的研究环境，不是运行依赖。
- 不上传凭据、会话数据库、实验运行日志、依赖或生成缓存。依赖锁文件在将来创建后应提交。

## 验证状态

已有15项关键事实的独立语义复核和29处源码锚点检查。它们是静态源码/文档证据；尚未安装产品依赖、执行应用测试、调用模型或完成 DSH 适配。

## License

采用 [MIT License](LICENSE)。允许商用、修改和再分发，但须保留版权和许可声明；不要求衍生项目开源，也不提供担保。上游资产仍须保留各自的版权与许可，详见[第三方声明](THIRD_PARTY_NOTICES.md)。
