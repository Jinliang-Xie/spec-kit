# 功能规格：基于专利文本的工序图谱（LLM + Python 工作流）

特性分支：`001-step-graph`
创建日期：2025-10-28
状态：草案
# 功能规范：专利步骤图谱提取与可视化

**功能 ID**: 001-step-graph  
**状态**: 草稿  
**版本**: 0.3.0  
**最后更新**: 2025-10-28  
**语言**: 中文  
**英文规范**: `spec.md`（权威版本）

---

## 执行摘要

构建一个模块化的 Python 管道，集成 LLM 能力，从专利 JSON 提取物中提取、表征、原子化和聚类技术"步骤"（原子过程操作），并构建多阶段 Neo4j 知识图谱，用于可视化、跨文档聚合和可复用性分析。

系统通过四个渐进阶段处理专利文本，每个阶段将结果持久化到 Neo4j 以进行增量可视化和人工验证，最终形成跨专利统一的步骤图谱。

---

## 目标与范围

### 主要目标

从专利衍生的 JSON 文件中提取结构化步骤实体及其关系，沿三个表征维度（能量、物质、信息）对其进行编码，将复合步骤原子化为不可分割的单元，在文档间聚类相似步骤，并在 Neo4j 中可视化每个阶段的知识图谱。

### 范围内

- **阶段 1（识别）**：提取步骤的输入/输出、设备、操作模式、反应/操作条件、环境；识别串行/并行关系；写入 Neo4j。
- **阶段 2（编码）**：沿三个维度（能量转换、物质转换、信息转换）使用类张量表示表征步骤；增强图谱。
- **阶段 3（原子化）**：在 LLM+证据约束下将复合步骤拆分为原子、不可分割的单元；保留溯源。
- **阶段 4（聚类）**：通过聚类/规范化聚合跨专利的相似步骤；创建跨文档统一图谱。
- **多阶段 Neo4j 可视化**：每个阶段使用 `run_id` 和 `stage` 标记写入增量结果，用于并排比较和调试。
- **CLI 优先的 LLM 集成**：Python 包装器调用 LLM CLI（例如 `openai`、`anthropic`），并进行日志记录、缓存和版本控制。
- **模块化 VS Code 终端工作流**：每个阶段可独立运行，在持久化工件之前需要用户确认。

### 范围外（v1 版本）

- 从 PDF 中提取专利文本（假定预提取的 JSON 输入）。
- 图谱的实时协作编辑。
- 高级 GDS 算法（例如 PageRank、社区检测）——推迟到未来迭代。
- 多语言 UI（CLI 和文档仅支持英文/中文）。
- 部署自动化（图谱更新的 CI/CD）。

---

## 宪章参考

本规范受项目宪章 `memory/constitution.en.md`（权威）和 `memory/constitution.md`（中文对应）治理。关键原则：

- **双通道输入契约**：通道 A（半结构化 JSON），通道 B（原始文本回退）。
- **阶段性 Neo4j 可视化**：四个阶段后强制持久化和可视化。
- **LLM 编排**：CLI + GitHub Copilot + Codex，统一日志记录/缓存。
- **稳健解析**：将 0/空/"null" 处理为未知；双轨原始/规范化参数。
- **图谱建模**：节点（Patent、Step、Parameter、Dimension、Cluster），关系（HAS_STEP、NEXT、HAS_PARAM、DERIVED_FROM、COMPOSED_OF、IN_CLUSTER、SAME_AS），遵循 Neo4j 最佳实践的约束和索引。

详细的输入模式和规范化启发式方法请参阅 `docs/data-profile.md`。

---

## 用户故事

### Epic 1：阶段 1 — 步骤识别

**US-1.1**：作为研究人员，我想从专利 JSON 文件的 `technical_process` 字段中提取结构化步骤，以便查看每个专利的有序操作序列。

- **验收标准**：
  - 按 `->` 分隔符拆分 `technical_process`；分配 `order_index`。
  - 在 Neo4j 中创建 `Patent` 和 `Step` 节点，属性包括：`patent_id`、`step.id`、`stage: "identification"`、`text_raw`、`run_id`。
  - 建立 `[:HAS_STEP]` 和 `[:NEXT]` 关系。
  - Neo4j 可视化显示专利内部步骤序列。

**US-1.2**：作为研究人员，我想提取参数（温度、压力、时间、RPM 等）并将其链接到步骤，以便分析操作条件。

- **验收标准**：
  - 解析 `technical_parameters` 对象；创建 `Parameter` 节点，属性包括 `param_name`、`raw_value`、`raw_unit`、`has_value`。
  - 当不存在显式链接时，使用 LLM 推断关联将参数链接到步骤（Q4=B）。当置信度较低时，实现应回退到基于证据的链接（见下文）。
  - 建立 `[:HAS_PARAM]` 关系，包含 `source_field`、`evidence`（文本片段）和 `link_confidence`（0-1）。
  - 根据宪章将 0/空/"null" 处理为未知。

理由：LLM 推断链接提高了数据集缺乏显式参数→步骤关联时的覆盖率；所有推断链接必须包含置信度分数和使用的文本证据。

**US-1.3**：作为开发人员，我想让阶段 1 管道作为 CLI 命令运行并将结果持久化到 Neo4j，以便在继续之前检查和确认输出。

- **验收标准**：
  - CLI：`python -m step_graph identify --input <json_dir> --output <artifacts_dir> --run-id <uuid> --neo4j-uri <uri>`
  - 日志：结构化 JSON，包含输入哈希、提示版本、模型、时间。
  - 缓存：以 `{input_hash, prompt_version, model}` 为键。
  - 最终 Neo4j 提交前的用户确认步骤。

### Epic 2：阶段 2 — 步骤编码/表征

**US-2.1**：作为研究人员，我想沿三个维度（能量、物质、信息转换）使用类张量表示表征每个步骤，以便在语义上比较步骤。

- **验收标准**：
  - LLM + 模式约束提取生成转换编码（例如 `energy_from_form→to_form`、`matter_transformation`、`info_transformation`）。
  - 创建可选的 `Dimension` 节点（例如 `energy:electrical->thermal`、`matter:liquid_heating`）。
  - 使用 `encoding_vector` 或 `transformation_tags` 增强 `Step` 节点。
  - 建立 `[:HAS_DIMENSION]` 关系。
  - Neo4j 可视化按主导转换类型显示步骤颜色编码。

**US-2.2**：作为开发人员，我想让阶段 2 输出标记为 `stage: "encoding"` 并链接到阶段 1 步骤，以便比较识别与编码结果。

- **验收标准**：
  - 阶段 2 更新现有 `Step` 节点或创建新属性字段。
  - 所有节点/关系使用与阶段 1 相同的 `run_id` 标记。
  - 提供 Cypher 查询模板以按 `stage` 过滤。

### Epic 3：阶段 3 — 步骤原子化

**US-3.1**：作为研究人员，我想将复合步骤拆分为原子、不可分割的单元，同时保留溯源，以便实现聚类的一致粒度。

- **验收标准**：
  - LLM 识别多维或复合步骤；将其拆分为原子单元。
  - 新 `Step` 节点标记为 `stage: "atomicized"`。
  - `[:COMPOSED_OF]` 关系将原子化步骤链接到原始复合步骤。
  - `[:DERIVED_FROM]` 关系追溯到 `Patent`。
  - 通过 `[:NEXT]` 保留 `order_index` 和串行/并行关系。

**US-3.2**：作为开发人员，我想记录带有拆分理由的原子化，以便审计和完善拆分规则。

- **验收标准**：
  - `[:COMPOSED_OF]` 包含 `split_reason` 属性。
  - 日志记录拆分前后的步骤计数和使用的 LLM 提示。

### Epic 4：阶段 4 — 步骤聚类/规范化

**US-4.1**：作为研究人员，我想将跨专利的相似步骤聚类到可复用的规范组中，以便识别常见流程模式。

- **验收标准**：
  - LLM + 相似度评分识别等效步骤。
  - 创建 `Cluster` 或 `CanonicalStep` 节点，属性包括 `cluster_id`、`cluster_method`、`run_id`。
  - 建立 `[:IN_CLUSTER]` 或 `[:SAME_AS]` 关系，可选 `similarity_score` 属性。
  - Neo4j 可视化显示跨专利统一图谱，突出显示聚类。

**US-4.2**：作为开发人员，我想通过 CLI 标志配置聚类阈值和方法，以便尝试不同的策略。

- **验收标准**：
  - CLI：`python -m step_graph cluster --method <llm|embedding|hybrid> --threshold <float> --run-id <uuid>`
  - 记录聚类参数和指标（例如聚类计数、平均相似度）。

### Epic 5：多阶段可视化和回滚

**US-5.1**：作为研究人员，我想在 Neo4j Browser 中并排可视化所有四个阶段，以便比较和调试管道输出。

- **验收标准**：
  - 所有节点/关系使用 `run_id` 和 `stage` 标记。
  - 提供 Cypher 模板：`MATCH (s:Step {run_id: $runId, stage: $stage}) RETURN s`。
  - 文档包含每个阶段的示例查询。

**US-5.2**：作为开发人员，我想按 `run_id` 回滚特定运行，以便清理失败或实验性运行。

- **验收标准**：
  - CLI：`python -m step_graph rollback --run-id <uuid>`
  - 执行：`MATCH (n {run_id: $runId}) DETACH DELETE n`。
  - 记录已删除的节点/关系计数。

---

## 功能需求

### FR-1：输入解析（双通道）

- **FR-1.1**：支持通道 A（半结构化 JSON），字段包括：`technical_process`、`technical_parameters`、`innovation_object`、`application_field` 等（参见 `docs/data-profile.md`）。
- **FR-1.2**：通过 LLM 基于模式约束的提取支持通道 B（原始文本回退）。

注释（通道 B 触发器）：

- 默认行为：优先使用通道 A（如果可用）。当用户传递 `--force-lane-b`（Q3=C）或预处理明确标记输入对通道 A 不可用时，调用通道 B。实现应记录何时使用通道 B。

理由：显式 `--force-lane-b` 避免意外回退并保持行为确定性。

- **FR-1.3**：规范化参数：将 0/空/"null" 映射为未知；将 `value:"1"/"0"` 解释为布尔值 `has_value`。
- **FR-1.4**：按中文/英文分隔符拆分列表字段（例如 `application_field`）；使用 `lang` 标记。

### FR-2：步骤识别（阶段 1）

- **FR-2.1**：按 `->` 拆分 `technical_process` 为有序步骤；分配 `id`、`order_index`、`text_raw`、`source_field`、`run_id`。

注释（步骤 ID 策略）：

- 选择策略：步骤 ID 使用 UUID v4（Q2=A）。每个阶段创建的每个步骤都接收一个新的 UUID。实现必须持久化一个映射表，链接跨阶段的 ID（例如 original_step_id -> atomicized_step_ids），以保留溯源和跨阶段可追溯性。

理由：UUID 避免意外冲突并简化幂等性；映射保留版本/阶段之间的血缘关系。

- **FR-2.2**：从 `technical_parameters` 提取参数；创建双轨属性（原始 + 规范化）。
- **FR-2.3**：将 `Patent`、`Step`、`Parameter` 节点写入 Neo4j；建立 `[:HAS_STEP]`、`[:NEXT]`、`[:HAS_PARAM]`、`[:DERIVED_FROM]`。
- **FR-2.4**：将所有阶段 1 节点标记为 `stage: "identification"`。

### FR-3：步骤编码（阶段 2）

- **FR-3.1**：LLM 沿三个维度（能量、物质、信息转换）表征步骤；输出符合模式的结构化 JSON。
- **FR-3.2**：使用 `encoding_vector`、`transformation_tags` 创建/更新 `Step` 节点。
- **FR-3.3**：可选地创建 `Dimension` 节点；建立 `[:HAS_DIMENSION]`。
- **FR-3.4**：将增强的节点/关系标记为 `stage: "encoding"`。

注释（维度编码模式）：

- 选择模式：每步骤结构化对象（Q1=B）。每个步骤编码必须包含三个维度的结构化对象。示例：

```json
{
  "dimensions": {
    "energy": {"type": "mechanical->thermal", "magnitude": 0.73},
    "matter": {"type": "mass_transfer", "magnitude": 0.12},
    "information": {"type": "control_signal", "magnitude": 0.05}
  },
  "encoding_vector": [/* 可选嵌入向量 */]
}
```

理由：`magnitude` 是归一化的浮点数（0-1），必须由一致的评分/聚合方法产生；幅度评分规则的设计和验证是阶段 2 实现的一部分。

### FR-4：步骤原子化（阶段 3）

- **FR-4.1**：LLM 识别复合步骤；在证据约束下拆分。

注释（原子化标准）：

- 原子化决策规则结合启发式方法（Q5 = A+B+C 组合，C 作为最后手段权威）：
  1. 优先自动规则：当步骤描述明确包含多个维度转换（首选单维度）或多个显式动词/动作（A/B）时拆分。
  2. 使用基于模式的规则（动词+对象启发式）和 LLM 建议提出拆分。
  3. 如果自动启发式方法存在分歧或置信度较低，升级到人在回路审查；人类判断（C）是最终权威，应谨慎使用。

理由：自动启发式方法最大限度地减少手动工作，同时在模糊性高时提供清晰的升级路径。

- **FR-4.2**：创建标记为 `stage: "atomicized"` 的新 `Step` 节点；通过 `[:COMPOSED_OF]` 链接，包含 `split_reason`。
- **FR-4.3**：通过 `[:NEXT]` 关系保留串行/并行顺序。

### FR-5：步骤聚类（阶段 4）

- **FR-5.1**：计算原子化步骤之间的相似度（基于 LLM 或嵌入）。默认阈值 = 0.85（可通过 CLI `--threshold` 配置）。
- **FR-5.2**：创建 `Cluster` 节点；建立 `[:IN_CLUSTER]` 或 `[:SAME_AS]`，包含 `similarity_score`。
- **FR-5.3**：标记为 `stage: "clustered"`。

### FR-6：Neo4j 图谱模型

- **FR-6.1**：实现唯一性约束：`patent_id`、`step.id`、`cluster_id`。
- **FR-6.2**：在以下字段上创建索引：`run_id`、`stage`、`order_index`、`param_name`、`cluster_method`。
- **FR-6.3**：使用 `MERGE` 进行幂等写入；每事务批量 500-10K 节点/关系。
- **FR-6.4**：仅参数化查询（防止 Cypher 注入）。
- **FR-6.5**：避免 >1KB 字符串作为直接属性；使用引用或分块。

### FR-7：LLM 集成

- **FR-7.1**：CLI 优先调用（例如 `openai`、`anthropic` CLI 或本地包装器）。
- **FR-7.2**：结构化 JSON 日志：`{run_id, stage, agent, model, prompt_id, input_hash, output_summary, cache_hit, cost_estimate, timestamp}`。
- **FR-7.3**：缓存以 `{input_hash, prompt_version, model}` 为键；存储为 JSONL。

注释（缓存策略：失效和 TTL）：

- 选择策略（根据 Q8 的助手决策）：基于版本的失效，带可选 TTL。当 `prompt_version` 更改时，缓存条目失效。此外，早于 30 天的缓存条目应过期。必须提供手动 `--clear-cache` CLI 标志以强制失效。实现可以使用基于大小的 LRU 驱逐作为优化。

理由：基于版本的失效确保提示更改时的可重现性；TTL 限制过时条目，同时仍从缓存中受益。

- **FR-7.4**：模式约束生成；通过正则/单位/范围检查验证；失败时回退到"未知 + 证据"。

### FR-8：模块化 CLI 工作流

- **FR-8.1**：每个阶段公开 CLI 子命令：`identify`、`encode`、`atomize`、`cluster`。
- **FR-8.2**：标志：`--input`、`--output`、`--run-id`、`--neo4j-uri`、`--model`、`--temperature`、`--cache`。
- **FR-8.3**：最终 Neo4j 写入前的用户确认（可选 `--yes` 跳过）。

注释（确认 UX）：

- 选择 UX（Q7=A）：CLI 暂停，打印简洁摘要（专利计数、创建/更新的步骤、链接的参数、估计的 LLM 调用和成本）并提示 `[Y/n]` 继续。包含 `--preview` 标志将摘要保存到 `<output_dir>/<run_id>/preview.txt` 以供离线审查。

理由：简单、可脚本化，并且对终端优先工作流熟悉。

- **FR-8.4**：工件持久化到 `<output_dir>/<run_id>/<stage>/`，带版本化输出。

### FR-9：回滚和版本控制

- **FR-9.1**：CLI：`rollback --run-id <uuid>`。
- **FR-9.2**：记录删除计数并提供撤消警告。

---

## 非功能需求

### NFR-1：性能

- **NFR-1.1**：阶段 1 在标准笔记本电脑上处理 100 个专利（平均每个 10 个步骤）<5 分钟（不包括 LLM 延迟）。
- **NFR-1.2**：Neo4j 批量写入：每事务 500-10K 节点/关系；监控内存。
- **NFR-1.3**：索引查询对典型过滤器（`run_id`、`stage`）<1 秒内返回结果。

### NFR-2：可靠性

- **NFR-2.1**：幂等写入（MERGE）；重新运行相同 `run_id` 更新而非重复。
- **NFR-2.2**：优雅的 LLM 故障处理：记录错误，跳过步骤，继续部分结果（用户决定重试或中止）。

注释（错误处理策略：部分失败）：

- 选择策略（Q9=A）：如果阶段 1 处理专利失败（格式错误的输入或不可恢复的错误），完全跳过该专利并继续处理其他专利。阶段 2-4 仅对成功处理的专利操作。对于瞬态 LLM 或网络错误，在标记步骤为失败并跳过之前实现重试（3 次尝试，指数退避）。

理由：跳过失败的专利避免阻塞批处理运行，同时保留故障的可见性以供后续补救。

### NFR-3：可维护性

- **NFR-3.1**：模块化 Python 包：`parsers/`、`llm/`、`neo4j/`、`cli/`。
- **NFR-3.2**：解析器和规范化器的单元测试（核心逻辑 >80% 覆盖率）。
- **NFR-3.3**：使用模拟 LLM 和测试 Neo4j 实例的端到端管道集成测试。

### NFR-4：安全性

- **NFR-4.1**：仅参数化 Cypher 查询（无字符串插值）。
- **NFR-4.2**：通过环境变量或 `.env` 文件提供凭证（不硬编码）。
- **NFR-4.3**：输入验证：拒绝格式错误的 JSON、超范围参数。

### NFR-5：可观察性

- **NFR-5.1**：所有阶段和 LLM 调用的结构化 JSONL 日志。
- **NFR-5.2**：日志轮换和保留策略（例如 30 天）。
- **NFR-5.3**：可选的 Prometheus 指标导出（节点/关系计数、LLM 延迟）。

---

## 数据模型（Neo4j）

### 节点标签和属性

- **Patent**（专利）
  - 属性：`patent_id`（唯一，字符串）、`source_file`（字符串）、`lang`（字符串）、`filing_date`（日期）、`run_id`（字符串）
  - 约束：`CREATE CONSTRAINT patent_id_unique FOR (p:Patent) REQUIRE p.patent_id IS UNIQUE`
  - 索引：`CREATE INDEX patent_run_id_idx FOR (p:Patent) ON (p.run_id)`

- **Step**（步骤）
  - 属性：`id`（唯一，字符串）、`stage`（枚举：identification/encoding/atomicized/clustered）、`order_index`（整数）、`text_raw`（字符串）、`source_field`（字符串）、`run_id`（字符串）、`encoding_vector`（可选列表）、`transformation_tags`（可选对象）
  - 约束：`CREATE CONSTRAINT step_id_unique FOR (s:Step) REQUIRE s.id IS UNIQUE`
  - 索引：
    - `CREATE INDEX step_run_id_idx FOR (s:Step) ON (s.run_id)`
    - `CREATE INDEX step_stage_idx FOR (s:Step) ON (s.stage)`
    - `CREATE INDEX step_order_idx FOR (s:Step) ON (s.order_index)`

- **Parameter**（参数）
  - 属性：`param_name`（字符串）、`raw_value`（字符串）、`raw_unit`（字符串）、`normalized_value`（浮点数/null）、`unit_std`（字符串）、`normalization_version`（字符串）、`confidence`（浮点数 0-1）、`has_value`（布尔值）、`run_id`（字符串）
  - 索引：`CREATE INDEX param_name_idx FOR (p:Parameter) ON (p.param_name)`

- **Dimension**（维度，可选）
  - 属性：`dimension_name`（字符串）、`taxonomy_version`（字符串）、`run_id`（字符串）

- **Cluster**（聚类）
  - 属性：`cluster_id`（唯一，字符串）、`cluster_method`（字符串）、`run_id`（字符串）
  - 约束：`CREATE CONSTRAINT cluster_id_unique FOR (c:Cluster) REQUIRE c.cluster_id IS UNIQUE`
  - 索引：`CREATE INDEX cluster_method_idx FOR (c:Cluster) ON (c.cluster_method)`

### 关系类型和属性

- **[:HAS_STEP]**：Patent → Step
  - 属性：`order`（整数，可选）

- **[:NEXT]**：Step → Step（文档内序列）
  - 属性：`order_delta`（整数，可选）、`parallel`（布尔值，默认 false）

- **[:HAS_PARAM]**：Step → Parameter
  - 属性：`source_field`（字符串）、`evidence`（字符串，文本片段）

- **[:DERIVED_FROM]**：Step → Patent
  - 属性：`source_field`（字符串，例如 "technical_process"）

- **[:COMPOSED_OF]**：Step → Step（原子化 → 原始复合）
  - 属性：`split_reason`（字符串）

- **[:HAS_DIMENSION]**：Step → Dimension（可选）

- **[:IN_CLUSTER]**：Step → Cluster

- **[:SAME_AS]**：Step ↔ Step（等价/规范化）
  - 属性：`similarity_score`（浮点数 0-1）

---

## I/O 契约（原则级别）

### 阶段 1：识别

- **输入**：符合通道 A 模式的 JSON 文件目录（参见 `docs/data-profile.md`）。
- **输出**：
  - `<output_dir>/<run_id>/stage1/steps.jsonl`（每行一个步骤及元数据）
  - `<output_dir>/<run_id>/stage1/parameters.jsonl`
  - Neo4j：Patent、Step、Parameter 节点及关系

### 阶段 2：编码

- **输入**：阶段 1 工件 + Neo4j Step 节点
- **输出**：
  - `<output_dir>/<run_id>/stage2/encodings.jsonl`（步骤 ID + 维度向量）
  - Neo4j：更新的 Step 节点，带编码属性；可选 Dimension 节点

### 阶段 3：原子化

- **输入**：阶段 2 工件 + Neo4j Step 节点
- **输出**：
  - `<output_dir>/<run_id>/stage3/atomized_steps.jsonl`
  - Neo4j：新原子化 Step 节点 + COMPOSED_OF 关系

### 阶段 4：聚类

- **输入**：阶段 3 工件 + Neo4j 原子化 Step 节点
- **输出**：
  - `<output_dir>/<run_id>/stage4/clusters.jsonl`（聚类 ID + 成员步骤 ID + 相似度矩阵）
  - Neo4j：Cluster 节点 + IN_CLUSTER/SAME_AS 关系

注释（跨阶段真实来源）：

- 选择协调模式（Q10=C）：双写。阶段必须同时写入 JSONL 工件（用于可重现性、审计和重新运行）和 Neo4j 图谱（用于查询和可视化）。系统将 Neo4j 视为主要可查询图谱，将 JSONL 工件视为可重现的溯源/日志存储；如果需要，任何一个都可以用于重建另一个。

理由：双写通过 Neo4j 提供快速探索，同时保留可审计、可重放的 JSONL 记录。

---

## 成功标准

1. **端到端管道**：从原始 JSON 到阶段 4 Neo4j 可视化的完整运行，无需手动干预（不包括确认步骤）。
2. **阶段性可视化**：所有四个阶段可视化可在 Neo4j Browser 中访问，通过 `run_id` 和 `stage` 清晰区分。
3. **参数提取**：`technical_parameters` 中 >90% 的显式参数正确解析并链接到步骤。
4. **原子化质量**：20 个专利样本的人工审查显示 >85% 的复合步骤正确拆分。
5. **聚类一致性**：人工审查确认 >80% 的聚类步骤在语义上相似（领域专家验证）。
6. **性能**：阶段 1 处理 100 个专利（不包括 LLM 延迟）<5 分钟。
7. **回滚**：成功回滚 `run_id`，零残留节点/关系。

---

## 待解决问题

1. **LLM 模型选择**：每个阶段使用哪些模型（GPT-4、Claude 3、特定领域微调）？
2. **多语言支持**：如何处理 `technical_process` 中的中英文混合文本？
3. **参数规范化规则版本控制**：谁批准 `normalization_rules_version` 的更新？
4. **可视化工具**：Neo4j Browser 足够还是需要自定义 UI（Bloom、GraphXR）？

注释：以下待解决问题已通过利益相关者澄清解决并编码到本规范中：

- 编码维度分类（原 #2）——已解决：带 `magnitude` 字段的结构化维度对象（参见 FR-3 注释）。
- 原子化护栏（原 #3）——已解决：组合启发式 + 人在回路升级（参见 FR-4 注释）。
- 聚类方法（原 #4）——已解决：默认阈值和方法可配置；默认阈值设置为 0.85（参见 FR-5）。

---

## 假设和约束

### 假设

- 用户可以访问 Neo4j 实例（本地或云）。
- 输入 JSON 文件格式良好且符合通道 A 模式（参见 `docs/data-profile.md`）。
- LLM API（OpenAI、Anthropic）可访问且适当限速。
- 可用于交互式开发的 VS Code 与 GitHub Copilot 或 Codex。

### 约束

- 需要 Python 3.12+。
- 约束语法和索引功能需要 Neo4j 5.x。
- LLM 配额限制可能限制处理；缓存缓解重复调用。
- 初始重点：电池/电化学领域专利（可稍后推广）。

---

## 参考资料

- **宪章（英文）**：`memory/constitution.en.md`
- **宪章（中文）**：`memory/constitution.md`
- **数据概况（英文）**：`docs/data-profile.md`
- **数据概况（中文）**：`docs/data-profile.zh-CN.md`
- **Neo4j 数据建模**：https://neo4j.com/docs/getting-started/data-modeling/
- **Cypher 手册**：https://neo4j.com/docs/cypher-manual/current/
- **Neo4j 约束**：https://neo4j.com/docs/cypher-manual/current/constraints/
- **GraphAcademy**：https://graphacademy.neo4j.com/courses/modeling-fundamentals/

---

## 附录：示例 Cypher 查询

### 按阶段查询步骤

```cypher
// 阶段 1：识别
MATCH (p:Patent)-[:HAS_STEP]->(s:Step {stage: "identification"})
WHERE s.run_id = $runId
RETURN p.patent_id, s.id, s.order_index, s.text_raw
ORDER BY s.order_index

// 阶段 2：编码
MATCH (s:Step {stage: "encoding", run_id: $runId})-[:HAS_DIMENSION]->(d:Dimension)
RETURN s.id, s.transformation_tags, collect(d.dimension_name) AS dimensions

// 阶段 3：原子化
MATCH (s_atom:Step {stage: "atomicized", run_id: $runId})-[:COMPOSED_OF]->(s_orig:Step)
RETURN s_atom.id, s_orig.id, s_atom.text_raw

// 阶段 4：聚类
MATCH (s:Step)-[:IN_CLUSTER]->(c:Cluster {run_id: $runId})
RETURN c.cluster_id, c.cluster_method, collect(s.id) AS members
```

### 按运行 ID 回滚

```cypher
MATCH (n {run_id: $runId})
DETACH DELETE n
```

### 按阶段计数节点

```cypher
MATCH (s:Step {run_id: $runId})
RETURN s.stage, count(s) AS step_count
ORDER BY s.stage
```

---

**规范结束**

参考：
- 宪章（英文为准）：`.specify/memory/constitution.en.md`（中文对照：`.specify/memory/constitution.md`）
- 样例数据（当前不在仓库）：用户提供的专利 JSON 与 CSV

## 1. 范围与阶段性交付物（与宪章一致）

阶段 1：工步识别（按专利）
- 从专利 JSON/文本抽取候选工步（初版），包含：输入/输出、设备、模式、环境/参数、以及顺序/并行关系。
- 交付：Neo4j 中的专利内图谱（Patent、Step 与关系）。

阶段 2：工步编码（统一刻画）
- 将工步映射到三大维度：能量、物质（传递/反应）、信息（采集/处理）。原子化之前允许多值。
- 交付：维度节点与 HAS_DIMENSION 关系；第二次可视化。

阶段 3：工步原子化（拆分/细化）
- 将多值/复合动作（或条件）的工步拆分为原子工步；保持顺序/并行语义与溯源关系。
- 交付：原子工步集合、DERIVED_FROM/COMPOSED_OF 溯源；第三次可视化。

阶段 4：聚类/同一化（跨文档）
- 聚类或 SAME_AS 标注相似原子工步，形成可复用的“标准工步”；去重融合。
- 交付：跨专利汇总图；第四次可视化。

超出范围：不做产品化 UI 或权限；算法阈值在 plan/ADR 中定义。

## 2. 用户场景与测试

用户故事 1（P1）：单专利流程检查
- 作为分析人员，我加载一个专利 JSON，生成阶段 1/2/3 图谱；我可以在 Neo4j Browser 中查询工步、参数与关系。
优先级原因：是后续阶段的基础，可用 MVP。
独立测试：对单 JSON 跑流程；核对节点/关系计数与样例正确性。
验收：
1. 给定一个有效 JSON，当我跑完阶段 1..3，则可见 Patent→HAS_STEP→Step；PRECEDES/并行；HAS_PARAM 与 HAS_DIMENSION 存在。
2. 给定多值维度的工步，当原子化执行后，复合工步被原子工步替代，并有 DERIVED_FROM 链接。

用户故事 2（P2）：跨专利对比
- 作为分析人员，我加载两个专利并可视化共享的标准工步。
独立测试：两个 JSON 产出跨文档的聚类/SAME_AS 关系。
验收：SAME_AS 或 IN_CLUSTER 关系存在，重复减少。

用户故事 3（P2）：溯源追踪
- 作为审计人员，我需要从任意原子工步追溯到原始文本片段与参数原始值/单位。
验收：每个参数保留 raw_text/raw_value 与 normalized_value/unit；边上记录 evidence_ref。

边界情形
- 单位缺失或范围不明确
- 并行与顺序的判别
- 复合工步导致的拆分爆炸
- 不同章节的冲突数值

## 3. 功能需求

FR-001：支持分阶段独立执行（stage-1..4），每阶段产物可持久化落盘。
FR-002：支持摄取最小契约的专利 JSON/或文本（doc_id、文本块、语言、时间戳）。
FR-003：抽取 Step 实体（step_id、source_ref、text_snippet、输入/输出（可选）、设备、模式、参数、关系）。
FR-004：按能量/物质/信息三维对工步编码，并建立 HAS_DIMENSION 到 Dimension。
FR-005：执行原子化，将复合工步拆分并记录 DERIVED_FROM/COMPOSED_OF。
FR-006：将环境/操作参数（温度、压力、时间、气氛、转速、流量、浓度等）作为 Parameter 以 HAS_PARAM 关联到 Step。
FR-007：写入 Neo4j 采用幂等 MERGE，带批次/版本标记，支持按批/版本回滚。
FR-008：跨文档聚类/同一化，创建 SAME_AS 或 IN_CLUSTER，不丢失溯源。
FR-009：记录全链路溯源：prompt、模型/参数版本、normalization_rules_version、source_ref、raw_text/raw_value。
FR-010：模块化、VS Code 终端可操作；阶段产物在用户确认后保存到项目目录下。

需澄清：
- FR-011：最小 JSON 输入契约超出 doc_id 与 blocks 的字段（title/pub_date/section types/language/figure refs）[澄清]
- FR-012：并行关系表达（显式 PARALLEL_WITH 或同层表示）[澄清]
- FR-013：三大维度的初始子维度与扩展机制 [澄清]
- FR-014：原子化的爆炸控制（最大拆分数、启发式）[澄清]

## 4. 关键实体（与技术无关）

Patent：id、title（可选）、pub_date（可选）、source_ref
Step：step_id、atomic_flag、source_ref、text_snippet（可选）
Dimension：name、subtype（可选层级）、description
Parameter：name、value（标量/范围/集合+限定词）、unit（规范单位）、raw_text、raw_value、unit_original、method、confidence、normalization_rules_version、source_ref
Cluster/CanonicalStep：id、label、centroid（概念）

关系：
HAS_STEP（Patent→Step）
PRECEDES/NEXT（Step→Step）
HAS_DIMENSION（Step→Dimension）
HAS_PARAM（Step→Parameter）
DERIVED_FROM/COMPOSED_OF（Step↔Step）
SAME_AS/IN_CLUSTER（Step→Cluster/CanonicalStep）

## 5. 输入/输出契约（原则级）

输入（专利 JSON 最小）：
- doc_id: string
- blocks: [ { id, text, section_type?, page?, lang?, meta? } ]
- 可选：title、pub_date、ipc/cpc、figures

输出（Neo4j）：
- 节点：Patent、Step、Dimension、Parameter、Cluster/CanonicalStep
- 边：HAS_STEP、PRECEDES/NEXT、HAS_DIMENSION、HAS_PARAM、DERIVED_FROM/COMPOSED_OF、SAME_AS/IN_CLUSTER
- 每阶段持久化产物：`.artifacts/<stage>/<doc_id>/` 下的结构化 JSON 快照和日志

## 6. 成功标准（可度量、技术中立）

SC-001：阶段 1..3 在样本集上无错误完成；每阶段生成可查询图谱与落盘产物。
SC-002：≥95% 的参数完成单位规范化，同时保留 raw_text/raw_value。
SC-003：原子化后复合工步被有效拆分且不丢溯源；审阅者可追溯。
SC-004：跨文档聚类产生非空的标准工步集，重复减少（首版可接受定性评审）。

## 7. 非目标/约束

- 本迭代不做产品化 UI 或权限系统。
- LLM 集成优先 CLI（Python 环境），API 兜底但需遵守同一契约。
- 不在本规格固化算法与阈值，相关内容在 plan/ADR。

## 8. 待澄清问题

- 参数词典 v0：受控名称与规范单位？谁维护？如何扩展？
- 原子化限流：最大拆分数、启发式，以及在歧义时的“部分原子化”标记？
- 并行语义：显式 PARALLEL_WITH 还是层级顺序表达？
- 维度分类法：初始子维度清单与变更治理？
- 缓存/日志：CLI 调用的默认落盘路径与保留策略，绕缓存策略？

## 9. 验收清单（规格签收）

- [ ] 阶段与交付物与宪章一致
- [ ] 实体与关系覆盖 Patent/Step/Dimension/Parameter/Cluster + 必要关系
- [ ] 输入/输出契约（原则级）完整
- [ ] 非目标与约束明确
- [ ] 待澄清问题已列出，供 /speckit.clarify 与计划使用
