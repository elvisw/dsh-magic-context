# 规划阶段的补充源码核查

在四问题研究完成后，为编写阶段0计划追加了以下只读核查。它们不属于原四问的15文件预算，不计入原报告的15项复核或29处锚点数字，也没有执行运行测试。

## DSH：原生结果替换已有示例

固定提交 `fcc0ad462172522aaf5c93a7b2e01f6b9f0b0159`：

- [compaction-tool-result-pruner实现](https://github.com/deepseek-ai/deepseek-harness/blob/fcc0ad462172522aaf5c93a7b2e01f6b9f0b0159/packages/compaction/compaction-tool-result-pruner/src/index.ts)扫描当前surface，保留原工具结果data与调用身份，只替换message.content。替换携带单节点surfaceOp和sourceEventSeqs；前置compaction/prune事件记录shadow-price。
- [对应fixture](https://github.com/deepseek-ai/deepseek-harness/blob/fcc0ad462172522aaf5c93a7b2e01f6b9f0b0159/packages/compaction/compaction-tool-result-pruner/tests/tool-result-pruner.spec.ts)检查原结果仍在日志、rich-block顺序、Unicode码点边界、幂等收敛与重建消息一致。这些是现有断言，不是本机通过记录。
- [SessionStore.flush](https://github.com/deepseek-ai/deepseek-harness/blob/fcc0ad462172522aaf5c93a7b2e01f6b9f0b0159/packages/core/session/src/index.ts#L1341-L1357)是持久性检查入口；必须使用已进入store的Session，等待全部监听器并传播失败，返回值表示是否有监听器参与。计划把false也视为无法证明持久化。
- [core文档](https://github.com/deepseek-ai/deepseek-harness/blob/fcc0ad462172522aaf5c93a7b2e01f6b9f0b0159/docs/subsystems/core.md#agentpre-step--waterfall)说明agent/pre-step是请求派生前的waterfall，接收Agent、消息、turn/step、signal和modelSelection；调用next保留当前消息。计划拟在此检查明确重建边界，但适配运行可行性仍待验证。

## DSH：真实加载和SDK不能被模拟替代

- [插件发布文档](https://github.com/deepseek-ai/deepseek-harness/blob/fcc0ad462172522aaf5c93a7b2e01f6b9f0b0159/docs/user/develop/basic/publish.md)区分bundle与profile；共享宿主包依赖和外部链接的解析位置会影响实例身份。只在独立Context里构造服务，不能证明真实profile加载成功。
- [CLI参考](https://github.com/deepseek-ai/deepseek-harness/blob/fcc0ad462172522aaf5c93a7b2e01f6b9f0b0159/apps/cli/reference/README.md)说明DSH_HOME与profile、patch层次，以及dump也可能初始化profile文件。from-default-profile会替换现有目标目录，不能当成无害的只读初始化命令。
- [SDK client文档](https://github.com/deepseek-ai/deepseek-harness/blob/fcc0ad462172522aaf5c93a7b2e01f6b9f0b0159/packages/sdk/client/README.md)描述DeepSeekHarness、run、resumeSession和close。显式dshBin能选择runtime；否则SDK默认解析同版本DSH。客户端env在提供时替换子环境，不是自动安全地继承。
- [SDK包清单](https://github.com/deepseek-ai/deepseek-harness/blob/fcc0ad462172522aaf5c93a7b2e01f6b9f0b0159/packages/sdk/client/package.json)版本为0.2.2-rc.1，并直接依赖DSH；[根清单](https://github.com/deepseek-ai/deepseek-harness/blob/fcc0ad462172522aaf5c93a7b2e01f6b9f0b0159/package.json)声明Node >=24.9.0。计划据此隔离SDK驱动依赖并固定到本机CLI，不能让SDK附带runtime替代目标悄悄通过。

## MC：局部资产提取不等于完整核心移植

固定提交 `33fb6ac9a39c8bb299776b6b514ea0815a15a225`：

- [pending-drop-eligibility.ts](https://github.com/cortexkit/magic-context/blob/33fb6ac9a39c8bb299776b6b514ea0815a15a225/packages/plugin/src/hooks/magic-context/pending-drop-eligibility.ts)中的pendingDropVeto检查保护集合和thinkingDropProtected；该文件其余部分依赖存储与保护策略。计划只提取纯函数并记录转换，不把整文件当成零依赖模块。
- [storage-tags.ts](https://github.com/cortexkit/magic-context/blob/33fb6ac9a39c8bb299776b6b514ea0815a15a225/packages/plugin/src/features/magic-context/storage-tags.ts)依赖SQLite、上游harness与其他内部策略，不能仅因函数名相似便宣称可直接载入DSH。
- [tag-owner-id.ts](https://github.com/cortexkit/magic-context/blob/33fb6ac9a39c8bb299776b6b514ea0815a15a225/packages/plugin/src/shared/tag-owner-id.ts)区分位置后缀和内容派生后缀，进一步说明稳定owner不等于当前数组索引。

## 许可

已直接读取两端固定提交的LICENSE：MC为MIT，版权行2025 Ufuk Altinok；DSH为MIT，版权行2026 DeepSeek。用户委托为本项目选开源协议后选择MIT。具体声明见[THIRD_PARTY_NOTICES](../../THIRD_PARTY_NOTICES.md)，不推定全部第三方依赖同为MIT。

上述观察支持计划分解，不证明跨存储协议、真实缓存收益、Windows冷恢复或完整插件已实现。
