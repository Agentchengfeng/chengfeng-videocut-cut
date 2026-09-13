---
name: chengfeng-videocut-cut
description: 剪辑中文口播原素材：逐词转录、词典修字出修字表、先句段通览远距离重复与多次 take，再逐词五轮找口误与重复、汇总表与重复句子表、打开 Studio 让用户复核、复盘沉淀用户偏好与词典。只产出一份已复核的删词账本，不切媒体、不做字幕、不做分镜动画。用户说剪口播、处理口误、生成口播基础素材、继续剪口播，或确认卡回传 action=return_cut_review 时使用。不要用于执行物理剪切、导出剪后视频、单独安装、单独打开工作台或口播分镜成片。
---

# 剪口播

先完整读取[共享接入规则](references/shared/plugin-access.md)：先识别用户已有工作台，核实本次能力；没有可用工作台时才询问是否安装 chengfeng-videocut。选其他工作台须转用其已验证方法；下方命令、账本和审核流程只适用于本产品，不能原样套用。不把缺 MCP 当成业务不可用。

从真实视频或受支持的 WAV 到**一份人工复核过的删词账本（Cuts + EDL）**，不修改原素材。七步一线：

```text
0  接入  先选已有工作台；选本产品后核验Runtime CLI、显式endpoint和服务器projectId
1  建档     真实媒体 → 对应 Product 导入（转录、暂存、建档）→ 读回状态
2  修字     词典把字改对（通用一本 + 用户一本）→ 出修字表
3  删词     读偏好与规则 → 句段通览找重复 take → 剩余完整逐词流五轮扫描
4  汇总     删词汇总表 + 重复句子表 → 按表自读校验
5  审核     全量提交 → 两张表呈给用户 → 打开 Studio 亲自复核
6  复盘交棒  对比提案与终版 → 归档三抽屉 → 报告四件事
```

**不切媒体、不导出成片**：Skill 做语义判断与编排，不物理剪切或覆盖原素材；Product
导入或预览可能产生派生媒体，这不等于剪后成片已导出。产品 Runtime 是项目、Cuts、EDL 和
Studio 状态的唯一写入者。

**用户只在三处说话**：同源项目选继续还是重来（仅剪过时）、Studio 复核、完成确认。
工作台选择、缺失输入、安装、费用与额外权限另按实际需要询问，不受上述三处限制；其余不重复打断。**每步的完成判据是它的产出物**：没出表，不算完成。

先读取并执行 [业务 Skill 的阶段合同](references/shared/business-workflow-contract.md)。
本文步骤与合同阶段的对应：1=preflight+Product state readback，5=proposal+Product CAS，
审核=project-level review binding。合同后三个阶段（确认、执行、验收）属于导出 Skill。

各条规矩的事故来历在 [事故簿](references/lessons.md)——规矩在正文，故事在那边。

## 0. 同一 Runtime CLI

先读[Runtime CLI](references/shared/runtime-cli.md)和[业务请求](references/shared/existing-workflow.md)。workbench connect核验显式endpoint；所有读取和提交带同一服务器projectId，不要求MCP、不启动第二服务、不转客户端registry。安装授权不受业务对话次数约束。

## 1. 建档与读取

已有项目直接workbench workflow-get/cuts-get/edit-list-get，不重ASR。新WAV经workbench ingest-start：先授权云ASR费用，绑定真实sourcePath、expectedSourceSha256、无Product输出的taskDirectory和operationId；随后ingest-status精确查原操作，成功后回读同projectId与三个revision。unknown/notfound不换ID再付费。

新视频HTTP建档尚非上述WAV接口能力；只在同Runtime专用本地project ingest已验证与目标registry、权限和原子导入合同一致时另行使用，否则报告能力缺口。不得调用裸transcribe、预写transcript.json、project.json、猜转录输出或传--output绕过Product。无云ASR报告missing_cloud_transcription_adapter，不回退本地转录。没有真实媒体不造占位物。

## 2. 修字：把字改对，出修字表

先读[共享AI词典](references/shared/ai-term-dictionary.md)和已有用户dictionary.md，用户词典优先；根据完整真实词流匹配，出原词→改成→上下文的修字表。词典是每次转录必过闸门，但不凭模糊相似自动替换专名。只有作者稿和上下文确认时才补改；无法确认就报告，不猜。

以workbench transcript-correct提交{operationId,expectedRevision,corrections:[{wordId,text}]}，expectedRevision来自本轮transcriptRevision，已有授权后加--confirmed。只改文字，不改时间、词数、wordId；只换写法不换意思，真说错又重说归删词。先呈修字表，写后复核回读及受影响字幕，保留人工修改，不宣称旧自动dictionary/align全部已迁到HTTP。

中文口语按原文保留：说「叉」「大模型」就是实际说法；是否采用专名正式显示写法依据用户词典和上下文，不擅自改变意思。词典不看上下文也可能改错，看到错误先修正判断和词典，不在其他文件补丁式覆盖。旧稿没有标点时不能假装原转录已有标点。

字幕与剪辑读同一标点。旧转录缺标点不能靠猜测补时间；regroup/align若不在workbench commands中不发明接口，仅另行核实同Runtime的专用本地CLI或报告缺口。修字后重新取得完整同版本playback再开始判断。

## 3. 删词：先句段通览，再逐词五轮

**干什么**：先通览完整句段，找远距离重复、多次重录和废弃 take，判断“哪一遍该留”；
再对剩余完整逐词流扫描五轮，确定“具体删哪些词”。刀口与停顿不由 Agent 猜毫秒数。

先读材料：

```json
["workbench","playback","--api-base","<origin>","--project","<projectId>","--file","<分页请求JSON>","--json"]
```

返回的是一页播放顺序稿，而不是可截断的大 JSON。由唯一协调者串行读取 `stream`；只要
`page.nextCursor` 不是 `null`，就继续调用同一命令并加
请求JSON的`cursor: page.nextCursor`（首轮`{limit:64}`）；每一页必须满足 `page.startIndex` 等于上一页的
`page.endIndex`，且 `transcriptRevision`、`editListRevision` 一直相同。只有读到
`nextCursor=null` 才算拿到完整输入。若收到 `revision_conflict / playback_cursor_stale`，
说明逐词稿或账本在中途变了：丢弃本轮句段索引和全部逐词记录，从第一页重扫，禁止拼接两种版本。

再用你的读文件工具读 `$HOME/.chengfeng-videocut/cut-preferences.md`（用户偏好，
他说过的算数；文件不存在就是新用户，跳过）。

判据在 [语义删除规则](references/semantic-deletion.md)（原则 + 判例）。
教材优先级：**用户偏好 > 用户判例 > 通用判例**。

### 第一层：通览所有句段

协调者用同一冻结版本的 `stream`，按已有标点、明显停顿和播放断点建立临时句段索引，
保留首尾 `wordId`、播放顺序和时间。它只用于阅读，不写项目文件，不成为另一份播放状态。
完整扫完所有句段再建立远距离重复组：表达任务、说了几遍、哪遍完整、各版差异、拟保留与拟删
take、对应语音 `wordIds` 和风险。关键词变化须结合上下文判断是重录修正还是新增信息；
不能只比较相邻句，也不能仅凭词不同就保留或删除。整句、长段、分叉与关键名词变化仍按高风险复核。
此层不调用 `workbench cuts-put`；具体规则见语义删除规则的“先句段，后逐词”。

### 第二层：对剩余完整逐词流做五轮

第一层拟删整段仅为临时排除项；每轮仍审视其余完整逐词流，保留 gap、`removedSpeech`、
播放断点及拟删段的位置证据，不能用缩短输入代替完整阅读。每遍只带一个判据：

```text
第 1 轮  重复          句间重复、句内重复（最客观：两遍相同）
第 2 轮  残句与改口     说一半断掉、换个说法重说
第 3 轮  口误重说       说错的数字、专名、词，后面重说对的
第 4 轮  英文卡壳       词头 / 字母 / 音节重复
第 5 轮  口头禅与语气词  最主观，按用户偏好的密度与容忍度来
```

每轮动作固定：带上第一层重复组 → 按本轮类型查判例 → 找到一处记一行（wordIds + 类型 + 风险）→
**不定夺别的类型**。第一轮回读并核对整句重复组，把“保哪遍”落实为精确语音 `wordIds`；
后面轮次遇到已标记的词直接跳过，冲突留到汇总解决。

规矩：

- **判断只用 `workbench playback` 的播放顺序，不许自己从 `transcript.json` 拼**——
  「说了两遍」靠播出来相邻，自己拼会把跳号读成缺内容而误否（见事故簿）
- 唯一协调者读完并冻结同一轮全部页面、完成句段通览后，可以把这份完整版本与重复组并行交给子 Agent 扫五类
  问题；**子 Agent 只能返回候选行**（稳定 `wordIds`、类型、风险、上下文），不能翻页、
  调用 `workbench cuts-put`、写项目账本或各自交提案。协调者去重、处理重叠并生成唯一一份全量候选；
  版本变化就丢弃所有子结果并整轮重扫
- 删除只有「删除 / 未删除」两态；AI 原因不形成「建议删除」第三态
- 口误、重复、残句默认删前保后；长句、整句、分叉重说必须高风险复核
- 保留所有 gap 证据，只提交真正要删的语音 `wordIds`，不把 gap ID 塞进语义提案强压停顿。
  自然停顿不超过 `300ms` 原样保留是产品策略目标，不是本 Skill 已验证的执行事实；
  停顿与新接缝按当前实际版本化策略和 EDL 回读验收，缺失或不一致就报告，不手改 EDL。

## 4. 汇总：两张表，表出来才动手

**干什么**：句段重复组与五轮的行去重、处理重叠后合并成两张表和唯一全量提案——这是本步的产出物，也是给用户看的东西。

**删词汇总表**（逐条删词决定）：

```text
| # | 时间   | 类型   | 删除              | 剩下读作          | 风险 | 依据      |
|---|-------|--------|------------------|------------------|-----|----------|
| 1 | 00:12 | 重复   | 这个功能呢         | 这个功能它其实      | 低  | 判例 R2   |
| 2 | 02:31 | 改口   | 比如我就让他跟踪    | 比如我让grok跟踪    | 高  | 判例 R5   |
```

**重复句子表**（说了多遍的句子，逐词表看不出「录了几遍、保哪遍」，而这正是用户
最想核对的）：

```text
| 句子（开头）      | 说了几遍 | 保了哪遍        | 删了哪几遍   | 风险            |
|-----------------|--------|---------------|------------|-----------------|
| 因为今天有什么…   | 3      | 第 3 遍 00:08  | 第 1、2 遍  | 高——整句级删除，核对上下文  |
| 多数Agent能搜…   | 2      | 第 2 遍 04:40  | 第 1 遍     | 高——两遍说法有出入，重点听 |
```

然后**语义自读**：按表把删后的全文按播放顺序通读一遍，读不通的行撤销。
这仅是语义复核：AI 自己再看一遍，不是拿去问用户；拿不准 = 不列（两态）。
它不证明尾音已删净、词头完整或接缝自然；声音边界按下方提交后检查独立报告。

## 5. 审核：到人工审核时才打开 Studio

**干什么**：全量提交候选，把两张表呈给用户，然后打开产品审核页让用户亲自复核。

```json
["workbench","cuts-get","--api-base","<origin>","--project","<projectId>","--json"]
["workbench","cuts-put","--api-base","<origin>","--project","<projectId>","--file","<请求JSON>","--confirmed","--json"]
```

候选只引用稳定 `wordIds`：

```json
{
  "expectedRevision": "<Cuts原revision>",
  "operationId": "<本次新操作ID>",
  "cutWordIds": ["word-12", "word-13"],
  "reasons": [
    { "wordIds": ["word-12", "word-13"], "kind": "repeat", "risk": "low" }
  ]
}
```

提交规矩：

- 为提交后接缝复核，在本轮工作上下文保留提交前 Product 回读的 EDL 与 revision，不写第二份
  活动账本；优先使用 Product 已有的版本化变更回执。若是接手已提交任务且拿不到前版，
  明确变化范围未知，不从当前段数推断本次检查总数或全覆盖。
- 候选是**本轮判断的完整结论，不是增量**——`workbench cuts-put` 是替换语义，交增量会把上一轮
  语义删词静默丢掉（见事故簿）；提交候选时禁止读取或手工合并
  `initialization.baselineCutWordIds`（产品自动并入停顿基线）
- 继续旧项目时，先用当前 Cuts/reasons 和上次留存提案核对仍有效的已确认语义删除，再与本轮新增候选合并；
  `removedSpeech` 不会因为本轮听不到就自动恢复，人工已恢复的词也不能按旧提案重新删。
  不盲拷全部 `cutWordIds` 混入停顿基线；无法分清既有语义选择时报告缺口，不以空集替换。
- `reasons` 落盘，如实写；当前合同每项为 `wordIds`、非空 `kind`、`risk=low|high`，不是未定义的枚举。
  产品会裁掉提到未删词的部分；已确认旧理由保留当前有效部分，缺字段不编造
- 不直接写 `cut-selection.json`、`project.json` 或事件日志
- `workbench cuts-get.data.revision` 是 Cuts 的 revision，`workbench workflow-get.data.revision` 是项目的，
  **禁止混用**
- 提交成功后必看 `noLongerCut`（本次不再删的词数）：不为零且非有意撤回，先核对是否误交增量；
  必要修正须按当前版本完整重读、重审后生成新全量提案，不盲目重交。`"unknown"` 不是重提授权；
  未知提交保留原 operationId 与输入，按共享合同查回，不换 ID 再写
- workbench cuts-put未知提交先只读workbench cuts-get/workflow-get/edit-list-get。
  当前没有专门的cuts operation查询命令；仅当前值吻合不证明原操作成功。若不能唯一确认，
  报告“提交结果待核实”，不编造查询接口，不以再次写入 代替查询，也不自动重试
- 随后立即再次 `workbench workflow-get` + `workbench cuts-get`，确认 `cut_review_ready` 与三个 revision；
  不一致即停，绝不直接写 JSON 或自动覆盖
- **提交后把 `<提案文件>` 留在任务目录**——第 6 步复盘要用（reasons 在用户编辑后
  会被产品裁掉，事后从账本反查不可靠）
- 每个稳定版本只有协调者能执行**一次** `workbench cuts-put` CAS。五轮不能各写一次；若 CAS 冲突，
  本轮即告失效，重新串行读完全部页面、重新扫描并合并全量候选后，才能对新 revision 发起
  新的一次 CAS；禁止把旧提案硬覆盖到新 revision

**提交后检查接缝**：按[边界冲突与声音复核](references/semantic-deletion.md#边界冲突与声音复核)
核对本次实际变化的接缝、诊断能力和已知异常。结构回读成功不能替代声音检查；能力缺失或
检查不全须说明，已知边界冲突不能当成已修复。不要为了补检查而改媒体、重转录或反复提交。

然后**先呈表、再开页**：把两张表及声音检查的覆盖范围、未解决异常发给用户，让他进 Studio 复核。

在同一显式origin打开既有Studio的`/#project/<projectId>`审核页；先核对该projectId与workflow/readback及页面capability。CLI connect不返回studio.url，不编造返回字段。

随后用 Codex 内置浏览器读取同一已核验origin 的
`/chengfeng-videocut-capabilities.json`，验证其中声明 `koubo` 顶层视图；不要再调用
任何 Plugin 私有脚本或用户系统浏览器。

规矩：

- 只有服务/项目/capability核验通过才用Codex内置浏览器打开同一项目URL，然后停止自动推进，
  等用户划词、恢复、保存
- 打开前把目标URL的 `#project/<projectId>` 与 readback 的 `projectId` 严格比对（不是假设connect返回productUrl），
  并绑定 `stage=cut_review_ready` 与三个 revision——门禁 `ok=true` 只证明产品面能力，
  不能替代项目级绑定；任何不一致重新 readback
- `studio_capability_missing`：停止并说明版本不兼容，可建议 `$chengfeng-videocut-maintain` 的 Bug 反馈模式；
  禁止因 URL 带 `?view=koubo` 就认为新界面存在，禁止回退旧任务面板
- 不要把「打开工作台」当任务第一步；不访问旧 `review.html`、8898/8899；
  不控制 Studio DOM、不直接改媒体元素；不创建独立音频轨或占位字幕轨

## 6. 复盘交棒：用户批改的作业不许扔

用户复核完成后，先读回：

```json
["workbench","workflow-get","--api-base","<origin>","--project","<projectId>","--json"]
["workbench","cuts-get","--api-base","<origin>","--project","<projectId>","--json"]
```

**复盘**（过渡做法，产品的 `cuts diff` 命令发布后替换）：拿第 5 步留存的
`<提案文件>` 与读回的终版 `cutWordIds` 对比——

- **用户恢复的**（提案里有、终版没有）= AI 删错了。对照 proposal 里自己写的
  reason 问：错在哪一类？
- **用户补删的**（终版有、提案没有、也不在 `initialization.baselineCutWordIds`
  停顿基线里）= AI 漏了。问：这属于哪个类型，为什么没识别出来？
  （基线只在这里用于剔除停顿词，仍然禁止把它并进提交）

归档三个抽屉：

- 口味差异 → `~/.chengfeng-videocut/cut-preferences.md` 追加（Agent 维护的
  markdown，只追加与改写自己的节；文件不存在就创建）
- 专名错误 → `~/.chengfeng-videocut/dictionary.md` 追加（两列表格，追加前查重）
- 规则空白 → 偏好文件「待升级判例」节记一笔；同类第三次出现时向用户报告，
  建议升级进通用判例（人审后才动 skills 仓库）
- 零差异也要记：偏好文件项目记录一行「全盘采纳」

**报告四件事**：删了多少词、少了多少秒；当前 stage（应为 `cut_review_ready`）；
项目 / Cuts / EDL 三个 revision；复盘摘要（提案 N 条，采纳 X 恢复 Y 补删 Z，
主要分歧一句话）。按用户目标及已完成阶段交棒：仍需字幕或画面时转对应 Skill；
只有这些阶段已完成，或用户明确不需要时，才进入导出准备。交棒不自动授权导出或发布。

**不弹确认卡，不执行剪切**——物理剪切的确认卡属于导出 Skill，那张卡冻结的必须是
用户按下确认那一刻的 revision。

`return_cut_review`：先workbench connect和workflow-get重核同一身份与项目，再返回同一Studio继续复核；
不触发复盘，复盘只在最终交棒做一次。

报告分级：Product 结构化 revision 为 **API/readback PASS**；真实浏览器帧审核才是
**visual frame PASS**；没人实际听音一律 **human listening UNVERIFIED**，不得用播放、
DOM、截图或媒体探测替代。
发生过残音、截字或卡顿反馈时，报告还须注明反馈位置是否复测、同类接缝检查范围和剩余缺口；
单处修复、代码测试或 Skill 更新均不能升级为全片声音验收。

## 恢复与失败

- `revision_conflict`：重新读取状态，说明用户刚才的编辑，不自动覆盖
- `runtime_unhealthy`：不循环重装
- `service_identity_mismatch` / `service_port_conflict`：停止，不回退 foreground、
  不换端口、不杀未知进程
- 页面关闭但服务仍在：读取 workflow 后从当前状态续做，不新建项目
- 任何失败都不得把「账本已写」说成「已经剪好」；Product 预览派生文件也不证明成片已导出

附注：原素材与云ASR输出由Product管理，媒体真实、导入回读完整，是判断的前提。
