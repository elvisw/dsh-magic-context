# Stage0 Context Lifecycle Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 在隔离 Windows/DSH 会话中验证文本/JSON工具内容的稳定编号、排队卸载、持久重放和原文恢复，不交付完整插件。

**Architecture:** 复用固定 MC 资产和行为契约；DSH 适配器通过原生 tool/result replacement 改变 surface，不改冻结请求。原始历史由 DSH 日志保存；实验侧库保存稳定引用、缩减意图和执行状态。离线模型适配器驱动真实安装的 DSH，模拟测试不能代替该集成门槛。

**Tech Stack:** JavaScript ESM/JSDoc（仅验证包，不预定正式产品语言）、Node >=24.9.0、node:test、node:sqlite、DSH 0.2.2-rc.1、同版本 TypeScript SDK client。

**Spec:** [设计草案](../specs/2026-11-07-dsh-magic-context-design.md)，特别是阶段0验收边界。事实依据：[研究报告](https://github.com/elvisw/dsh-plugin-magic-context/blob/main/_bmad-output/initiative-dsh-magic-context/research-context-lifecycle/research-context-lifecycle.md)。

**Plan status:** draft / awaiting user review；仅计划作者自检完成，所有实施任务仍未执行。

## Authorization

- 用户已同意阶段0范围和编写验证计划；**本计划尚待审核，不构成实现、安装依赖或启动 DSH 的授权**。
- 完整 V1、最终存储位置、跨 harness 数据互通、正式禁用策略均未批准。
- 后续获准执行时仅使用合成数据和独立 DSH_HOME，不向日常 profile 安装。
- 用户已另行授权Git初始化及推送，仓库已建立；这不授权执行本计划。后续获得实施授权时，只提交任务列出的变更，不混入未审查的用户文件。

## Global Constraints

- MC 固定 v0.49.40 / `33fb6ac9a39c8bb299776b6b514ea0815a15a225`。
- DSH 源码基线 `fcc0ad462172522aaf5c93a7b2e01f6b9f0b0159`；目标为已安装 CLI `0.2.2-rc.1`。运行包须记录实际版本和入口，不假定与源码逐文件一致。
- 不使用 agent/request 或 llm/stream 改写正文；不引入必需的自定义消息投影解释器。
- 文本逐字符、参数 JSON 原始字符串逐字符恢复；不可重新执行工具或以摘要代替。二进制附件、推理块不在范围内，混合结果须明确拒绝卸载。
- 不连接真实模型、不读取已有凭据、不复制现有 MC 数据、不启动 Historian/embedding/UI。SDK 安装可能联网取包，不等于允许模型调用。
- 复用源码/fixture 保留 MIT 许可、提交、原路径、提取范围、摘要及本地差异。
- 较重的非研究规划子代理优先 deepseek-v4.1-flash，尤其开发和评审；替代须说明原因。

## Review Focus

1. 重复 call ID、多个兄弟工具和会话间相同编号不能串档（Task 2）。
2. flush 失败或进程在两个存储提交之间退出，不能重复应用或带病发送（Task 4）。
3. Unicode、CRLF、JSON空白、边界分页不能改变原文或无提示截断（Task 5）。
4. 错误/默认环境、凭据继承和路径逃逸不能触及日常数据或真实模型（Task 1/6）。
5. 插件禁用后冷加载、外部摘要已替换目标或取消，不能依赖缺失解释器或覆盖新历史（Task 3/4/6）。

## File Structure

以下均为未来创建文件；当前仅创建本文。

| 路径 | 单一责任 |
|---|---|
| `package.json`, `.gitignore` | 实验包、node:test命令；排除生成数据 |
| `vendor/magic-context/{LICENSE,manifest.json,pending-drop-veto.mjs}` | 上游纯保护判断的可追踪提取 |
| `src/stage0/contracts.mjs` | JSDoc契约与结构化错误码 |
| `src/stage0/journal.mjs` | SQLite身份、队列、操作日志 |
| `src/stage0/dsh-port.mjs` | 原始读取、原生替换、durable flush |
| `src/stage0/coordinator.mjs` | 保护、一次性模式、提交和对账 |
| `src/stage0/expand.mjs` | 按引用恢复与显式分页 |
| `src/stage0/plugin.mjs` | DSH工具注册和pre-step接入 |
| `scripts/stage0/{environment,run,child}.mjs` | 环境隔离、SDK驱动、故障注入 |
| `tools/stage0-driver/{package.json,client.mjs}` | 隔离安装SDK依赖并导出DeepSeekHarness，不污染宿主插件解析 |
| `tests/stage0/{environment,identity,port,coordinator,expand,integration}.test.mjs` | 对应边界测试 |
| `tests/stage0/fixtures/{scenarios,offline-adapter}.mjs` | 合成历史和确定性模型流 |
| `tests/stage0/fixtures/host-probe.mjs` | 实际宿主的原生Session断言与独立离线provider，禁用缩减插件时仍可运行 |
| `docs/stage0/{runbook,results}.md` | 可复现运行说明和真实结果 |

Task 1 → 2 → 3 → 4 → 5 → 6。阶段0仅覆盖保留调用配对、替换结果正文的固定模式；上游整对移除、LKG、全部缓存调度和自动缩减留待后续，不宣称等价移植完成。

## Shared Contracts

以下签名是实验模块之间的约定，不冒充 DSH 公共 API。以 JSDoc 记录；所有编号须为非负安全整数，tag 须为正安全整数。

```ts
type Origin = {
  sessionId: string; ownerSeq: number; callSeq: number;
  resultSeq: number; callId: string;
};
type RawTool = {
  origin: Origin; name: string; argumentsRaw: string;
  content: readonly { type: 'text'; text: string }[]; isError: boolean;
};
type TagRecord = { tag: number; raw: RawTool; digest: string };
type DropIntent = {
  opId: string; sessionId: string; tag: number; originalResultSeq: number;
  digest: string; marker: string;
  state: 'queued' | 'prepared' | 'applied' | 'blocked';
  headBeforePrepare: number | null; shadowSeq: number | null;
  replacementSeq: number | null;
};
type ApplyWindow = {
  rebuildAllowed: boolean; reason: 'explicit-stage0-boundary' | 'no-rebuild';
  protectedTags: ReadonlySet<number>; signal: AbortSignal;
};
type ApplyResult = { applied: number[]; pending: number[]; blocked: number[] };
type ExpandPage = {
  origin: Origin; name: string; argumentsRaw: string; isError: boolean;
  text: string; block: number; start: number; end: number;
  next: { block: number; offset: number } | null;
  complete: boolean;
};
```

`TagRecord.raw` 是读取结果，不表示侧库复制完整原文。侧库只存 Origin、digest 和决定；由 DSH 读取原文后组装返回。digest 为明确字段顺序的 `{name, argumentsRaw, content, isError}` JSON 的 SHA-256；不解析重写 argumentsRaw。

错误统一为 `Stage0Error(code, message)`：`ENV_UNSAFE`、`HOST_UNSUPPORTED`、`BAD_REF`、`RAW_MISSING`、`IDENTITY_MISMATCH`、`UNSUPPORTED_CONTENT`、`TARGET_STALE`、`DURABILITY_FAILED`、`RECOVERY_REQUIRED`、`ABORTED`。不得将错误返回为伪造的成功原文。

### Task 1: 隔离运行与宿主能力门槛

**Files:** Create `package.json`, `.gitignore`, `scripts/stage0/environment.mjs`, `tests/stage0/environment.test.mjs`, `tools/stage0-driver/{package.json,client.mjs}`, `docs/stage0/runbook.md`。

**Interfaces:** Produces `prepareTrial(baseDir: string): Promise<{home: string, cwd: string, env: Record<string,string>}>`；`probeHost(): Promise<{cliVersion: string, cliPath: string, nodeVersion: string}>`。

- [ ] **Step 1 — RED:** 测试 `rejects_existing_home_and_unsafe_paths`：拒绝已存在/默认 home、junction/symlink 逃逸；`builds_clean_child_environment`：只保留启动必需的 PATH、SystemRoot、TEMP/TMP 等白名单，设置唯一 DSH_HOME，不能继承 API key、代理、NODE_OPTIONS、DSH配置覆盖或项目 .env；子进程 cwd 为新空目录。用合成环境值测试，不读取真实秘密。

```js
await assert.rejects(() => prepareTrial('C:/Users/w_elv/.dsh'), { code: 'ENV_UNSAFE' });
const trial = await prepareTrial('C:/Users/w_elv/AppData/Local/Temp/opencode');
assert.notEqual(trial.home, trial.cwd);
assert.equal(trial.env.DSH_HOME, trial.home);
assert.equal(trial.env.DEEPSEEK_API_KEY, undefined);
assert.equal(trial.env.NODE_OPTIONS, undefined);
```
- [ ] **Step 2 — 验证失败:** `node --test tests/stage0/environment.test.mjs`；预期缺少导出或断言 FAIL，而不是网络/凭据错误。
- [ ] **Step 3 — 最小实现:** 实现上述接口。home 只能在 `C:/Users/w_elv/AppData/Local/Temp/opencode/dsh-mc-stage0-<UUID>/home` 下新建；拒绝重用。只查询 `dsh --version` 和入口，Node要求 >=24.9.0。实验包 private、ESM、engines.dsh 精确 `0.2.2-rc.1`，将dsh-tools与dsh-llm声明为同版本peer，不在根目录安装宿主副本。仅在tools/stage0-driver子包安装精确SDK版本0.2.2-rc.1；client.mjs导出其DeepSeekHarness，明确传入已安装CLI，不使用SDK默认runtime。驱动依赖不得成为plugin.mjs的祖先node_modules。
- [ ] **Step 4 — GREEN/能力门槛:** 重跑 Step 2。执行阶段才允许安装必要 SDK 依赖并锁定实际解析版本；不得自动批准安装脚本。核对已安装 SDK 的 dshBin/启动选项类型和 runtime peer 解析，记录与固定源码契约的差异；无法固定到目标 CLI 就停止，不换目标冒充通过。
- [ ] **Step 5 — 检查点:** 保存实际入口、版本、隔离策略和命令到 runbook；忽略 `.stage0/`、`node_modules/`、临时数据库。仅在已授权 Git 仓库中提交本任务文件，消息 `test: establish isolated stage0 host gate`。

重启只复用本次trial清单中已创建、校验真实路径的home，不再次调用prepareTrial，也不以reset命令覆盖profile。SDK驱动依赖安装命令为 `npm install --prefix tools/stage0-driver --ignore-scripts`，只在执行获准后运行；需要脚本时单独报告。根测试使用node内建模块，不要求根目录npm install。

### Task 2: 可追踪上游资产与稳定身份日志

**Files:** Create vendor三文件、`src/stage0/{contracts,journal}.mjs`、`tests/stage0/identity.test.mjs`、`tests/stage0/fixtures/scenarios.mjs`。

**Interfaces:** Consumes Origin/TagRecord/DropIntent；Produces `openJournal(path: string): Journal`，方法 `assign(raw: RawTool): TagRecord`、`lookup(sessionId: string, tag: number): {origin: Origin,digest: string}`、`queue(sessionId: string, tags: readonly number[]): DropIntent[]`、`list(sessionId: string): DropIntent[]`、`prepare(opId: string, marker: string, headBeforePrepare: number): void`、`recordShadow(opId: string, shadowSeq: number): void`、`applied(opId: string, replacementSeq: number): void`、`block(opId: string): void`、`close(): void`。

- [ ] **Step 1 — RED:** `stable_tags_survive_reopen` 断言重开同库编号不变；`owners_and_sessions_do_not_alias` 断言重复 call ID 的不同 owner、兄弟工具、另一会话不串档；`queue_is_idempotent_and_transactional` 断言重复入队只有一个 opId，任一未知 tag 使整批失败且不部分入队；缺失/越界/非整数为 BAD_REF。

scenarios.mjs固定导出rawA：sessionId=`s0`、ownerSeq=2、callSeq=3、resultSeq=4、callId=`same`、name=`echo`、argumentsRaw为包含空白的 `{ "x": 1 }`、单text块为 `A😀\r\n` 重复4097次、isError=false；rawB使用另一个owner/seq但复用same，rawOther使用另一会话s1。对应断言核心如下（tmpDb由每测试独立临时目录提供）：

```js
const journal = openJournal(tmpDb);
const first = journal.assign(rawA);
assert.notEqual(journal.assign(rawB).tag, first.tag);
journal.close();
const reopened = openJournal(tmpDb);
assert.equal(reopened.assign(rawA).tag, first.tag);
assert.throws(() => reopened.queue('s0', [first.tag, 999999]), { code: 'BAD_REF' });
assert.deepEqual(reopened.list('s0'), []);
```
- [ ] **Step 2 — 验证失败:** `node --test tests/stage0/identity.test.mjs`；预期接口未实现导致 FAIL。
- [ ] **Step 3 — 最小实现:** SQLite侧库放该 trial home 下，只存引用/digest/决定，事务分配每会话递增 tag；owner键包含 sessionId、ownerSeq、callSeq、resultSeq、callId，不使用 surface下标或单独callId。唯一约束保护同一原始结果及同一缩减操作。开启 foreign_keys、WAL、synchronous=FULL；不创建全产品schema或迁移现有库。
- [ ] **Step 4 — 复用资产:** 从固定 MC 的 `packages/plugin/src/hooks/magic-context/pending-drop-eligibility.ts` 提取完整 `pendingDropVeto` 函数；只删除 TypeScript 类型标注形成 mjs，不复制其无关存储依赖。manifest 记录提取符号、原路径、SHA、原文摘要与转换；保留原 LICENSE。scenarios 迁移 owner碰撞、tag/ordinal不同、兄弟工具的断言语义，并标明源fixture，不把 adapted fixture 冒充原样执行。
- [ ] **Step 5 — GREEN/检查点:** 重跑 Step 2，增加保护集合返回 protected、thinkingDropProtected 返回 active_thinking、其他返回 null 的资产对照测试；检查依赖中无 OpenCode/Pi SDK。条件提交消息 `feat: add stage0 identity journal and upstream provenance`。

### Task 3: DSH原始读取与原生结果替换

**Files:** Create `src/stage0/dsh-port.mjs`, `tests/stage0/port.test.mjs`；Extend scenarios。

**Interfaces:** Consumes Origin；Produces `createDshPort(ctx: object, session: object): DshPort`，方法 `origins(): readonly Origin[]`、`readRaw(origin: Origin): RawTool`、`inspect(intent: DropIntent): {kind: 'original'|'landed'|'stale', seq: number|null}`、`replace(intent: DropIntent): number`、`flush(): Promise<void>`、`messages(): readonly object[]`。origins只列出已完整配对的原始调用/结果，不把替换事件当新调用。replace只追加原生事件并返回新seq，flush才确认持久化；inspect必须核对来源和内容，不靠marker字符串猜测。

- [ ] **Step 1 — RED:** `raw_reader_ignores_shadowed_surface` 从原事件恢复，不能读到占位；`replacement_preserves_call_and_metadata` 对比 role、callId、错误身份、meta与原调用参数，仅content改变；`rejects_foreign_missing_mixed_and_stale_targets` 分别断言身份错误、来源缺失、非文本混合结果和已被其他压缩替换；`flush_requires_a_durability_listener` 断言 false/异常不被视为成功。

```js
assert.deepEqual(port.readRaw(rawA.origin), rawA);
assert.equal(port.inspect(intent).kind, 'original');
const replacementSeq = port.replace(intent);
assert.equal(port.inspect(intent).seq, replacementSeq);
assert.deepEqual(port.readRaw(rawA.origin), rawA);
await assert.rejects(() => portWithoutListener.flush(), { code: 'DURABILITY_FAILED' });
```

本段port/intent/portWithoutListener由测试使用同一rawA构造，后者只把flush监听器结果设为false；不能依赖真实用户会话。
- [ ] **Step 2 — 验证失败:** `node --test tests/stage0/port.test.mjs`；预期接口未实现 FAIL。
- [ ] **Step 3 — 最小实现:** 通过 session原始事件读取绑定owner/call/result；使用原事件完整data，仅替换message.content。原生替换调用为 `session.append('tool/result', data, {surfaceOp: {op:'replace', startSeq: seq, endSeq: seq}, sourceEventSeqs:[seq]})`。参照固定DSH pruner，紧邻替换前记录 compaction/prune shadow-price，由ctx.tokenMeter估算；不得将自定义投影解释器写入会话。flush必须调用 `await ctx.sessions.flush(session)`，false为DURABILITY_FAILED。
- [ ] **Step 4 — GREEN/语义验证:** 重跑 Step 2，单测核对适配器调用和错误传播。把DSH原生Session fixture交由Task 6的host-probe实际执行：原结果仍在snapshotEvents、派生消息含marker而不含原正文、重建Session得到相同消息、兄弟工具不变。本步的stub断言不冒充宿主通过。将上下游API差异记录到runbook。
- [ ] **Step 5 — 检查点:** 条件提交消息 `feat: adapt stage0 reductions to native dsh surface events`。真实宿主不接受上述替换或持久化能力不成立时停止，不改LLM loop。

### Task 4: 缩减协调、持久提交与恢复对账

**Files:** Create `src/stage0/coordinator.mjs`, `tests/stage0/coordinator.test.mjs`, `scripts/stage0/child.mjs`；Extend journal、port、scenarios。

**Interfaces:** Consumes Journal、DshPort、ApplyWindow与上游veto；Produces `queueDrop(journal: Journal, sessionId: string, tags: readonly number[]): DropIntent[]`、`applyPending(journal: Journal, port: DshPort, sessionId: string, window: ApplyWindow): Promise<ApplyResult>`、`reconcile(journal: Journal, port: DshPort, sessionId: string): Promise<void>`。Journal额外持久记录准备时的日志head、shadowSeq（若有）和replacementSeq供对账，更新在自身事务内完成。

- [ ] **Step 1 — RED:** `queue_does_not_mutate_surface`、`no_rebuild_or_protected_tags_stay_pending`、`cancelled_or_stale_target_never_applies`；断言无授权边界/保护/取消不追加替换。`frozen_marker_does_not_change_with_age` 和 `repeated_apply_is_a_noop` 断言模式字节不变、同一操作只有一次替换。测试两并发apply被每会话锁串行化。

```js
const tag = journal.assign(rawA).tag;
const before = port.messages();
queueDrop(journal, 's0', [tag]);
assert.deepEqual(port.messages(), before);
const result = await applyPending(journal, port, 's0', {
  rebuildAllowed: false, reason: 'no-rebuild', protectedTags: new Set(),
  signal: new AbortController().signal,
});
assert.deepEqual(result.applied, []);
assert.deepEqual(result.pending, [tag]);
```
- [ ] **Step 2 — 验证失败:** `node --test tests/stage0/coordinator.test.mjs`；预期FAIL。
- [ ] **Step 3 — 最小实现:** queued → prepared（冻结opId/tag组成的短marker和原始digest）→ 原生shadow/replacement → await durable flush → applied。只有明确的 `explicit-stage0-boundary` 允许首次修改；这是假定的实验重建点，不冒充完整上游缓存许可算法。缩减后必须确实更短，否则不应用。任何失败阻断该会话后续模型请求，取消不等于回滚已提交事件。

prepare前还要验证原始来源已成功durable flush、身份和digest一致、参数UTF-8不超过64KiB且所有结果块为text，确保存在受支持的恢复路径。不满足时明确拒绝，不先卸载再让ctx_expand报错；为超大参数及混合块增加“无替换事件”断言。
- [ ] **Step 4 — 故障测试:** 对prepared持久后、shadow追加后、replacement追加后、DSH flush后但journal.applied前分别注入失败。新进程以持久日志对账：原节点仍当前则可继续；完全匹配的已落地替换只补记applied；唯一匹配的孤立shadow只补替换、不再计价；来源不明或目标变化则blocked/TARGET_STALE，禁止覆盖。flush失败后，不凭内存inspect认定落地；必须冷读持久日志。用真实Windows子进程终止覆盖选定边界，禁止只靠内存对象重建宣称崩溃恢复。
- [ ] **Step 5 — GREEN/检查点:** 重跑 Step 2，断言每个恢复路径不重复shadow计价/替换、不遗失原文；无法唯一对账时测试应确认阻止发送而非伪造成功。Windows终止测试不得skip为通过。条件提交消息 `feat: make stage0 reductions replayable across commit failures`。

### Task 5: 文本原文恢复和显式分页

**Files:** Create `src/stage0/expand.mjs`, `tests/stage0/expand.test.mjs`；Extend scenarios。

**Interfaces:** Consumes Journal.lookup、DshPort.readRaw；Produces `expandTag(journal: Journal, port: DshPort, sessionId: string, tag: number, cursor?: {block: number, offset: number}): ExpandPage`。每页最多4096个Unicode码点，offset同样按码点；argumentsRaw完整返回，不重新序列化；工具参数超过64KiB时明确返回 UNSUPPORTED_CONTENT，阶段0不隐式截断它。上述限制是验证包保护值，不是正式产品承诺。

- [ ] **Step 1 — RED:** `expand_returns_original_without_undropping` 对比原始参数字符串、各文本块、错误标记和origin，确认缩减状态与旧surface不改变；`unicode_crlf_and_json_whitespace_roundtrip` 将全部页面按block/offset拼回，断言原字符序列相等且无替换字符；`tag_is_not_message_ordinal` 和 `dropped_sibling_does_not_alias` 使用Task 2 fixtures。

```js
const page = expandTag(journal, port, 's0', tag);
assert.equal(page.argumentsRaw, rawA.argumentsRaw);
assert.deepEqual(page.origin, rawA.origin);
assert.equal(Array.from(page.text).length, 4096);
assert.deepEqual(page.next, { block: 0, offset: 4096 });
assert.equal(journal.list('s0')[0].state, 'applied');
```

此测试setup先用Task 4把rawA的tag实际置为applied；其余页面拼接后必须与rawA.content[0].text严格相等。
- [ ] **Step 2 — 验证失败:** `node --test tests/stage0/expand.test.mjs`；预期FAIL。
- [ ] **Step 3 — 最小实现:** 先查持久身份，再读DSH原始事件并核对digest；禁止调用原工具、读取当前文件或改用当前surface。按文本块分页，返回精确block/start/end/next/complete，末页next=null。错误上下文不得包含其他会话原文。
- [ ] **Step 4 — GREEN/边界:** 增加 `page_boundaries_are_exact`，覆盖0、4095、4096、4097码点和多文本块；空结果返回complete=true；非法cursor为BAD_REF；超大参数、缺失档案、错误digest和图片混合块分别显式失败。重跑Step 2，断言失败不修改journal或Session。
- [ ] **Step 5 — 检查点:** 条件提交消息 `feat: recover exact stage0 tool text with bounded pages`。

### Task 6: 真实安装DSH的离线闭环与结论

**Files:** Create `src/stage0/plugin.mjs`, `scripts/stage0/run.mjs`, `tests/stage0/integration.test.mjs`, `tests/stage0/fixtures/{offline-adapter,host-probe}.mjs`, `docs/stage0/results.md`；Extend runbook与package.json。隔离overlay由run.mjs在trial目录生成，不写用户profile。

**Interfaces:** Consumes Tasks 1–5；Produces `runScenario(name: 'normal'|'restart'|'faults'|'disabled'): Promise<{status: 'pass'|'fail'|'blocked', evidenceDir: string}>`，以及DSH `apply(ctx)` 入口。包脚本 `test:stage0` 为 `node --test tests/stage0/*.test.mjs`，`validate:stage0` 为 `node scripts/stage0/run.mjs`。SDK启动显式指定已安装dshBin、隔离dshHome、cwd、env、patches及唯一 `stage0/mock` 模型路线；按实际安装SDK类型传参，不猜测不存在的字段。

- [ ] **Step 1 — RED:** `real_host_completes_reduce_expand_cycle` 断言真实DSH至少经历原始大结果、ctx_reduce排队、不允许重建时仍保留、允许重建后变小、ctx_expand返回原文；禁止用自写Session模拟器替代。`rejects_non_mock_routes` 断言所有其他provider/model立即拒绝；脚本化适配器只产生合成工具调用，不执行shell/文件操作。

```js
const result = await runScenario('normal');
assert.equal(result.status, 'pass');
const restarted = await runScenario('restart');
assert.equal(restarted.status, 'pass');
```
- [ ] **Step 2 — 验证失败:** `node --test tests/stage0/integration.test.mjs`；预期未实现的入口/断言FAIL；宿主无法加载则标记HOST_UNSUPPORTED，不当成产品测试成功。
- [ ] **Step 3 — 最小接入:** 声明Cordis依赖并通过ctx.tools.register/defineTool注册阶段0子集：ctx_reduce只接受逗号分隔正整数drop，ctx_expand只接受单个tag和可选分页cursor；其他上游参数明确拒绝。工具execute依据实际执行Agent定位会话，不读环境中的“最近会话”。在agent/pre-step调用next取得决定，只在enter、未取消且通过Task 4门槛时提交；保持messages原样，不修改已冻结请求。并发和拒绝决定的测试必须覆盖。

进入获准pre-step后先以port.origins/readRaw为受支持的原始结果journal.assign；重复扫描不新增tag，替换事件不能再次编号。不支持的结果保留原样并记录原因。offline-adapter使用固定fixture及已确认tag编排调用；本阶段不声称已验证真实模型理解缩减提示或自行选择该丢什么。
- [ ] **Step 4 — 全链路:** 使用SDK的run、resumeSession和close完成normal/restart；恢复后先reconcile，后允许模型请求。实际请求应被冻结、工具配对与source引用合法、未修改前缀一致；缩减重建请求记录explicit-stage0-boundary。ctx_expand新结果中出现原文是预期行为，不误判为旧位置被恢复。捕获未经归一化的结构化请求，不能删除差异字段来制造稳定性。
- [ ] **Step 5 — 故障与禁用:** faults运行Task 4选定边界的真实子进程终止和冷恢复；disabled在不加载缩减插件、保留离线fixture provider的情况下冷加载已缩减会话，验证Session可读、原日志可导出、不缺解释器。记录ctx_expand在禁用时不可用、不自动还原可见正文；这仅验证最低安全边界，不替用户选择正式产品的A/B禁用体验。
- [ ] **Step 6 — 总验证:** `npm run test:stage0` 与 `npm run validate:stage0` 均应退出0，且无required-case skip；进程退出、SQLite关闭、旧home不变。results逐项记录真实命令、版本、结果、证据路径；失败/环境阻塞照实写fail/blocked，不能用源码测试存在或模拟通过替代实际通过。
- [ ] **Step 7 — 检查点:** 仅提交源文件/运行说明/脱敏结论，不提交trial日志、数据库或依赖目录。条件提交消息 `test: prove stage0 lifecycle on isolated installed dsh`。返回用户评审是否继续阶段1，不自动做完整移植。

## Gate and Evidence Matrix

| 门槛 | 通过证据 | 失败后的动作 |
|---|---|---|
| G1 目标与隔离 | 实际CLI版本/入口；唯一home；凭据和真实模型拒绝测试 | 停止，不换runtime或读取日常配置 |
| G2 身份与资产 | 重开稳定编号；碰撞/兄弟工具断言；MIT与manifest | 修正本任务，禁止进入缩减 |
| G3 宿主持久变更 | 原文仍在日志；合法原生替换；真实flush与重建 | 报告宿主约束，不改冻结正文 |
| G4 部分提交恢复 | 选定终止点的冷读证据；无重复计价/替换；错误时禁止发送 | 无法唯一对账即停止回到设计 |
| G5 原文恢复 | 全部页面拼接与原始文本/参数一致；错误定位拒绝 | 不以摘要或重新执行降级 |
| G6 整体闭环 | 真实DSH离线请求、重启、禁用冷读和资源关闭 | 记录fail/blocked，不宣称可交付 |

正文缩减以UTF-8字节和Unicode码点分别记录，要求受控大结果缩减后严格更小；不设没有依据的收益百分比。前缀比较是结构化请求证据，不是HTTP逐字节或真实提供商缓存命中保证。延迟、恢复体积、累计请求体积记录基线；真实费用、命中率和全平台保证明确写not tested。

## Self-Review and Execution Handoff

- [x] 需求覆盖：FR-1/RULE-2/3/4对应Tasks 2–6；持久化和Windows失败边界对应Tasks 3/4/6。FR-2至FR-8不是本计划交付物，没有从总PRD删除。
- [x] 步骤检查：每项具备文件、接口、RED/GREEN命令、可判定断言和检查点；外部能力不满足时有停止门槛。
- [x] 接口检查：Origin与raw权威分开；replacementSeq仅在真实事件存在时记入；ExpandPage带分页与isError，未静默丢字段。
- [x] Review Focus：五项均已落到所属任务测试，不能只在最后报告里提及。
- [x] 比例检查：六项按独立可拒绝的交付边界切分，没有独立脚手架任务，也没有把代码实现全文塞进计划。

本节检查由计划作者自行执行，不另派子代理。审核通过后再选执行方式：子代理逐任务实现与独立评审，或主会话顺序实现、最后独立评审。无论选择哪种，较重开发/评审委派遵循AGENTS.md的模型成本规则。
