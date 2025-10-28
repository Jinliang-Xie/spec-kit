# 项目宪章（中文对照版）

说明：本文件为中文对照版；英文版为规范的唯一权威来源，见 `memory/constitution.en.md`。

## 目标与范围

将 LLM 集成到 Python 工作流，从专利相关 JSON 文本中识别并原子化"工步（Step）"，进行统一表征与聚类，并在 Neo4j 中构建可视化的"工序图谱（Step Graph）"。管线阶段：
- 阶段 1：识别（Identification）
- 阶段 2：编码/统一表征（Encoding / Unified Characterization）
- 阶段 3：原子化（Atomicization / Splitting & Refinement）
- 阶段 4：聚类与规范化（Clustering / Canonicalization）

输出要求：每一阶段可在 VS Code 终端中独立运行与检查，产物在用户确认后落盘保存，供下一阶段消费与审计。

### 阶段性可视化（必需）

- 每个阶段完成后必须写入 Neo4j 并生成对应可视化，用于多阶段对比、核验与调试。
- 可视化要求：
  - 阶段 1（识别）：Patent 节点 + Step 节点（初步识别）+ HAS_STEP/NEXT 关系；每个 Step 标注 `stage: "identification"`、`order_index`、`text_raw`。
  - 阶段 2（编码）：在阶段 1 基础上，Step 节点增加 `encoding_vector`/`dimension_tags` 等字段；可选增加 Dimension 节点与 HAS_DIMENSION 关系。
  - 阶段 3（原子化）：新增原子化后 Step 节点（标注 `stage: "atomicized"`）；通过 COMPOSED_OF 关系指向原始 Step；保留 DERIVED_FROM 指向 Patent。
  - 阶段 4（聚类）：新增 Cluster 节点或 CanonicalStep 节点；通过 IN_CLUSTER 或 SAME_AS 关联原子化 Step；可选增加相似度权重。
- 版本隔离：每次运行以 `run_id` 标注所有节点与关系，支持同一数据库内多版本并存与对比；提供 Cypher 查询模板按 `run_id` 过滤可视化。
- 参考文档：Neo4j 官方文档 https://neo4j.com/docs/ 中的 Cypher 查询、数据建模与可视化最佳实践。

## 输入契约（双通道）

- 通道 A（半结构化/整理版 JSON）：优先支持当前仓库下的专利领域整理版 JSON，其常见字段包括：
	- `technical_process`（字符串，使用 "->" 连接步骤）
	- `technical_parameters`（嵌套对象，含 `nominal_capacity`、`nominal_voltage`、`charge_rate`、`discharge_rate`、`power_density`、`energy_density`、`internal_resistance` 等，子字段常见：`<name>`、`unit`、`value`（"1"/"0" 标志）、`actions` 文本）
	- 其他上下文字段：`innovation_object`、`innovation_component`、`technical_effect`、`application_field`、`battery_type`、`electrochemical_battery_type`、`separator`、`sustainability`、`safety` 等。
	- 数据特性：可能存在数值 0 与空 unit 表示“未知/未提供”，`value` 为字符串（"1"/"0"）表示是否具备该参数，个别字段值可能为字符串 "null"。内容以中文为主，夹杂单位与符号。
- 通道 B（原始专利文本/通用 JSON）：保留支持来自专利原文/第三方平台的通用 JSON/文本，按“抽取→校验→结构化”流程渐进支持；如未提供结构化字段，则从正文抽取步骤与参数。

原则：输入解析器需鲁棒处理“0/空/\"null\"/中英文混排/逗号或顿号分隔列表”，并将语义不确定值映射为“未知（unknown）”而非数学零。

参见：数据特征详解文档 `docs/data-profile.md`（英文为准）与 `docs/data-profile.zh-CN.md`（中文对照），包含字段观察、异常与归一启发式，供后续步骤调用。

## 步骤识别与原子化

- 识别来源优先级：`technical_process` 明确给出→ 其次结合 `technical_effect`、`application_field` 等补充上下文；通道 B 则从正文抽取（LLM+规则）。
- 原子化规则（通道 A）：
	- 以 "->" 进行第一层切分，得到有序步骤序列；保留原顺序并赋予 `order_index`。
	- 对包含并列/重复/修饰信息的步骤，进行适度细化与去重，保留关键信号（材料、配比、温度、时间、电压/电流等）。
	- 对引用/循环/条件类语句，以“注记字段”保存，不强制展开。
	- 产出每个 Step 的最小信息：`id`、`order_index`、`text_raw`、`tokens`（可选）、`source_field`、`source_doc`、`provenance`。
- 通道 B 原子化：结合句法/模板与 LLM（受 Schema 约束）完成切分与精炼。

## 参数建模与归一

- 参数作为一等公民：在图中建 `Parameter` 节点，通过 `HAS_PARAM` 关联到 Step 或 Step 组；`actions` 文本作为溯源描述保留。
- 双轨并存（Raw/Normalized）：
	- 原始：`raw_text`、`raw_numeric`、`raw_unit`、`source_field`、`source_path`、`evidence`。
	- 标准：`normalized_value`、`unit_std`（SI 或行业常用）、`normalization_rules_version`、`confidence`。
- 0/空/"0"/"null" 处理：一律视为“未知”，仅在语义明确为数值零时才记为 0；`value:"1"/"0"` 字段映射为布尔型 `has_value`。
- 多语言与列表：`application_field` 等以中文分隔符（、/，）或英文逗号拆分为列表，保留 `lang` 标记。
- 参数字典 v0（可增量演进）：`nominal_capacity(mAh/Ah)`、`nominal_voltage(V)`、`charge_rate(C)`、`discharge_rate(C)`、`power_density(W/kg)`、`energy_density(Wh/kg)`、`internal_resistance(mΩ)`、`cycle_life(次)`、`self_discharge_rate(%)`、`operating_temperature_min/max(°C)`、`separator_thickness(mm)`、`porosity(%)`、`dimensions(length,width,height + unit)` 等。

## 图谱最小模型（Neo4j）

### 节点与标签（Labels）

- `Patent`：专利文档节点；属性至少包含 `patent_id`（唯一）、`source_file`、`lang`。
- `Step`：工步节点；属性至少包含 `id`（唯一）、`stage`（"identification"/"encoding"/"atomicized"/"clustered"）、`order_index`、`text_raw`、`run_id`。
- `Parameter`：参数节点；属性包含 `param_name`、`raw_value`、`raw_unit`、`normalized_value`、`unit_std`、`normalization_version`、`confidence`、`has_value`（布尔）。
- `Dimension`（可选）：维度/类别节点；属性包含 `dimension_name`、`taxonomy_version`。
- `Cluster`：聚类节点；属性包含 `cluster_id`、`cluster_method`、`run_id`。

### 关系类型（Relationship Types）

- `[:HAS_STEP]`：Patent → Step，表示专利包含工步；属性可选 `order`（序号）。
- `[:NEXT]`：Step → Step，同一文档内工步顺序链；属性可选 `order_delta`（跨度）。
- `[:HAS_PARAM]`：Step → Parameter，工步关联参数；属性可选 `source_field`（来源字段）、`evidence`（溯源文本片段）。
- `[:DERIVED_FROM]`：Step → Patent，原子化/聚类后溯源到原文档；属性 `source_field`（如 `technical_process`）。
- `[:COMPOSED_OF]`：Step → Step，原子化后的新 Step 指向被拆分的原始 Step；属性可选 `split_reason`。
- `[:HAS_DIMENSION]`（可选）：Step → Dimension，工步关联维度标签。
- `[:IN_CLUSTER]`：Step → Cluster，工步归属聚类。
- `[:SAME_AS]`：Step ↔ Step，对等/规范化关系（如同义步骤）；属性可选 `similarity_score`。

### 约束与索引（Constraints & Indexes）

- 唯一性约束（Uniqueness Constraints）：
  - `CREATE CONSTRAINT patent_id_unique FOR (p:Patent) REQUIRE p.patent_id IS UNIQUE`
  - `CREATE CONSTRAINT step_id_unique FOR (s:Step) REQUIRE s.id IS UNIQUE`
  - `CREATE CONSTRAINT cluster_id_unique FOR (c:Cluster) REQUIRE c.cluster_id IS UNIQUE`
- 索引（Indexes）：
  - 为高频查询字段创建索引：`run_id`、`stage`、`order_index`、`param_name`、`cluster_method`。
  - 示例：`CREATE INDEX step_run_id_idx FOR (s:Step) ON (s.run_id)`
  - 示例：`CREATE INDEX step_stage_idx FOR (s:Step) ON (s.stage)`
- 存在性约束（Property Existence，Enterprise）：可选对关键属性（如 `Step.stage`、`Patent.patent_id`）强制非空。

### 写入原则（参考 Neo4j 最佳实践）

- 幂等写入：使用 `MERGE` 而非 `CREATE`；对节点按唯一标识（如 `patent_id`、`step.id`）合并，避免重复。
- 批量事务：每批 500–10,000 节点/关系；使用 `UNWIND` + 参数化 Cypher；监控内存与事务日志。
- 参数化查询：所有 Cypher 查询必须使用参数（`$param`），避免 Cypher 注入与提升查询缓存命中。
- 版本隔离：每次运行的所有节点与关系标注 `run_id`（UUID 或时间戳）；支持同一数据库多版本并存与对比。
- 回滚支持：记录每个 `run_id` 的节点/关系数量与时间戳；提供删除指定 `run_id` 的 Cypher 模板：
  ```cypher
  MATCH (n {run_id: $run_id})
  DETACH DELETE n
  ```
- 属性规范：避免大字符串（>1KB）直接存节点属性；考虑外部存储 + 引用或分块存储。

### 参考与学习资源

- Neo4j 数据建模基础：https://neo4j.com/docs/getting-started/data-modeling/
- Cypher 查询手册（Cypher 5）：https://neo4j.com/docs/cypher-manual/current/
- 约束与索引管理：https://neo4j.com/docs/cypher-manual/current/constraints/
- GraphAcademy 数据建模课程：https://graphacademy.neo4j.com/courses/modeling-fundamentals/

## LLM 调用与可追溯

- Python 优先，CLI 调用（如 `openai/anthropic/…` CLI 或本地脚本）作为第一选择；所有 Prompt/Model/参数版本化记录。
- 严格日志与缓存：结构化 JSON 日志，包含输入摘要、模型与超参、输出摘要、耗时、费用估算；缓存键包含输入哈希与 prompt 版本。
- 反幻觉：以模式/schema 约束输出；对关键字段做正则/单位/范围校验；无法通过校验时回退为“未知 + 证据”。

### 三者协同：逐步 CLI + GitHub Copilot + Codex

- 逐步 CLI 为生产路径：每一阶段必须提供稳定的 CLI 子命令（文本/JSON I/O），支持 `--model`、`--temperature`、`--cache`、`--run-id` 等参数；任何 LLM 产出需经 CLI 执行、落盘与记录日志后方视为有效输入下一阶段。
- GitHub Copilot 为交互辅助：在 VS Code 中用于起草/重构代码与快速原型 Prompt；提交前需将稳定 Prompt/命令固化到代理目录（见 `AGENTS.md` 约定），并通过对应 CLI 子命令复现与校验输出。
- Codex CLI 为批量/脚本化调用：将常用任务固化为 `.codex/commands/*`（或等价目录），参数通过 `$ARGUMENTS` 传递；与管线 CLI 共享一致的日志/缓存规范（`run_id`、`input_hash`、`prompt_version`、`agent`、`model` 等）。
- 统一追踪与缓存：不论来自 CLI、Copilot 触发，还是 Codex CLI，统一写入 JSONL 日志与缓存；缓存键至少包含 `input_hash + prompt_version + model`；日志字段至少包含 `run_id`、`stage`、`agent`、`model`、`prompt_id`、`cache_hit`、`cost_estimate`。
- 审核闸门：IDE 侧生成的结果不得直接进入流水线下一阶段，必须通过对应 CLI 子命令验证与落盘通过后方可使用。
- 目录与约定：代理/助手相关命令与工作流文件按 `AGENTS.md` 的目录约定放置（如 `.github/prompts/`、`.codex/commands/` 等），便于跨工具复用与打包发布。

## 治理与语言策略

- 文档双语：英文版为规范的唯一权威来源；中文为对照与沟通辅助。两者需在版本上同步并注明差异。
- 变更门槛：对输入契约、参数字典、图谱模型的修改需提供“样本数据证据 + 影响评估 + 回滚方案”。
- 质量闸门：
	- 构建：脚本/依赖可重复安装与运行（文档化）。
	- 类型/格式：所有阶段产物通过 JSON Schema/校验器；单位/范围合规。
	- 测试：最小可用测试覆盖“步骤切分 + 2 个参数归一 + 图写入幂等”。

版本：0.2.0（草案） | 修订：根据 `IG_start/data/raw_patent_data` 最新样本校准了“输入契约（双通道）”、“0/空/\"null\" 解释规则”、参数字典 v0 与原子化规则。
