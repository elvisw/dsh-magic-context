# Fork 开发交接笔记

## 来源
- 相邻 `acceptance-windows-dsh-2026-11-09/issue-requeue.md`：#13 当前原位恢复缺陷、源码依据及两种不同契约。
- 相邻 `acceptance-windows-dsh-2026-11-09/issue-historian-recovery.md`：#14 已验证现象与摘要后独立原文检索建议。
- `../initiative-dsh-magic-context.md`：研究输入与当前实施边界。

## 已核对
- 实测插件为 `dsh-plugin-magic-context@0.11.1`，宿主为 `0.2.2-rc.1`；两者版本不能混写。
- #13 现有原位恢复会替换旧占位符，因此再次回收旧 tag 有意义；若采用新结果返回语义，应回收新结果的 tag。
- #14 不要求旧 tag 必须可用；持久化定位与原文保真是关键，不要求撤销整段摘要。
- 原版对照提交为 `17801ffd2f9635af1abae893e37914359110f2c1`，属于源码/测试核查，未另行执行原版验收。
- 工作区已有未提交的研究/验收材料、PRD/initiative 修改及 `.cortexkit/`；交接不能假设仅克隆本仓库就能取得当前材料。
- README 仍含早期独立移植路线和阶段0状态；本次添加交接入口与历史状态提示，不擅自重定产品范围或改写原始基线。

## 补充核对
- 验收报告及 REPRODUCE.md 已区分旧版缺陷证据与新语义验收。现有验证脚本依赖本轮固定会话、部分复现依赖本机临时实验目录，不是通用产品回归套件。
- 早期独立移植研究基线为 Magic Context `v0.49.40` / `33fb6ac9a39c8bb299776b6b514ea0815a15a225`；Issue 对照使用另一提交，不能混成同一次基线或已运行验收。
- 用户提供的相邻 fork 已只读核对：`main`、工作区干净、HEAD 为 `93ef5fe4a06a2c05f53e6a6225346919e860d74e`、package.json 版本 `0.11.1`，origin 对应 `elvisw/dsh-plugin-magic-context`，未配置名为 upstream 的 remote。此为当前本地快照，不保证远端未更新。
- fork 内未发现 AGENTS.md；现有 CONTRIBUTING.md 要求 npm、保留 package-lock.json、核心宿主中立、先写行为测试，且不提交生成物、私人数据或本机绝对路径。
- 未在 fork 安装依赖、运行测试、创建分支、写文件或更改 Git 配置。后续开发仍需用户授权和确认恢复契约。

## 文档校验与交付
- 交付 `handoff-fork-development.md`，含资料顺序、版本区分、两项问题的契约与拟议验收、贡献边界、跨机资料交付要求和只读启动提示。
- README、研究总索引和 initiative 已加入导航与历史范围提示，不改原始研究结论、证据或 PRD 需求。
- 校验覆盖 6 份文档、53 个本地链接，无缺失和行尾空白；从实际 fork 根目录可解析启动提示中的研究入口。git diff --check 通过。
- fork 工作区复核仍干净；未运行模型或产品测试。本次仅验证文档，不宣称 #13/#14 已修复或原版已经运行验收。
