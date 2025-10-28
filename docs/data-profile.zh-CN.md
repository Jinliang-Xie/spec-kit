# 专利 JSON 数据特征概览（中文对照）

日期：2025-10-28
数据目录：`IG_start/data/raw_patent_data`
范围：基于对 CN、US、EP、JP、WO 等多来源样本的代表性取样。英文版为规范的唯一权威来源，见 `docs/data-profile.md`。

## 取样清单（已审阅）

- WO-9713286-A1.json
- US-7611801-B2.json、US-2023136430-A1.json、US-2021242466-A1.json、US-11604523-B2.json
- EP-1110255-A1.json、EP-4354638-A1.json、EP-4071911-A1.json、EP-4102607-A1.json
- JP-S59127382-A.json、JP-2957756-B2.json、JP-H01144574-A.json
- CN-105162225-B.json、CN-116438709-A.json、CN-209767221-U.json、CN-213124660-U.json

说明：数据集规模更大，本文档将随覆盖面扩大而演进。

## 顶层字段（观测）

- `isRelated`（类布尔）：多为 `true`。
- `innovation_object`（字符串）：改进对象简述。
- `innovation_component`（字符串数组）：领域构件；偶见字面字符串 `"null"`。
- `technical_effect`（字符串数组）：技术效果，中文表述。
- `application_field`（字符串）：以中文标点分隔的领域列表。
- `battery_type`（字符串数组）：高层用途（动力/储能/其他）。
- `technical_process`（字符串）：以 `->` 连接的“步骤”文本；常见且对步骤识别最有价值。
- `electrochemical_battery_type`（字符串数组）：电化学体系；可能包含 `"null"`。
- `technical_parameters`（对象）：指标集合（见下一节）。
- `cycle_life`、`self_discharge_rate`、`operating_temperature`（对象）：含数值与单位。
- `dimensions`（对象）：`length`、`width`、`height`、`unit`；单位多样（mm、cm），0 频繁用作占位。
- `weight`（对象）：`value`、`unit`；常为 0 或空单位。
- `cathode_material`、`anode_material`、`electrolyte_type`、`electrolyte_solute`（字符串）：其中 `electrolyte_type` 偶见 `"null"`。
- `separator`（对象）：`base_material`、`coating`、`thickness{value,unit}`、`porosity{value,unit}`；大量 0。
- `sustainability`、`cost_optimization`、`safety`（对象）：子项含 `value`（"1"/"0"）与 `actions` 文本。

## `technical_parameters` 结构

单项参数常包含：
- 冗余名称键（如 `nominal_capacity.nominal_capacity`）。
- `unit`（字符串）：可为空。
- `value`（字符串标志）："1" 表示具备/关注，"0" 表示未具备/未提及；非数值量纲。
- `actions`（字符串）：解释/证据。
- 数值（可选）：如 `nominal_voltage: 4.5` 配 `unit: V`。

常见指标：`nominal_capacity`、`nominal_voltage`、`charge_rate`、`discharge_rate`、`power_density`、`energy_density`、`internal_resistance`、`energy_efficiency`。

## 数据特性与异常

- 占位零：数值字段大量为 `0` 且 `unit` 为空，语义应判作“未知/未提供”，非数学 0。
- 字符串标志：`value` 多为 "1"/"0"，应转为布尔 `has_value`。
- 字符串 `"null"`：出现在数组与字符串中，统一映射为 `unknown`。
- 单位：长度 `mm`/`cm` 混用；温度 `°C`/`℃` 并存；容量 `mAh`/`Ah`；倍率 `C`。
- 语言：中文为主；分隔符含 `、`、`，` 与英文逗号 `,`。

## 解析与归一（启发式）

1）步骤
- 若存在 `technical_process`，优先按 `->` 切分为有序 `Step`，保序记 `order_index`；保留 `text_raw`，再做精炼原子化。

2）参数
- 解析规则：
  - 对冗余名称键中的数值尝试读取（如 `nominal_voltage.nominal_voltage`）。
  - `value: "1"|"0"` 转布尔 `has_value`。
  - 若数值为 0 且单位为空，标注为 `unknown`，勿当作数值 0。
  - `actions` 作为溯源文本保留。
- 双轨存储：
  - 原始：`raw_text`、`raw_numeric`、`raw_unit`、`source_field`。
  - 标准：`normalized_value`、`unit_std`（SI/行业常用）、`normalization_rules_version`、`confidence`。
- 单位映射示例：
  - 容量 mAh/Ah（统一到约定量纲，记录原始单位）。
  - 温度 `℃` 等价 `°C`。
  - 长度 `cm` → `mm`（×10）。
  - 倍率 `C` 保持不变，值为浮点。

3）类别清洗
- 数组内的 `"null"`：移除或映射为 `unknown`。
- `electrolyte_type`：映射为“液态/固态/半固态/其他/未知”，同时保留原文。
- 列表类字符串（如 `application_field`）：以 `、`、`，`、`,` 拆分并去空白；加 `lang: zh` 标记。

## 参数字典 v0（观测 + 建议）

- nominal_capacity（mAh/Ah）
- nominal_voltage（V）
- charge_rate（C）
- discharge_rate（C）
- power_density（W/kg）
- energy_density（Wh/kg）
- internal_resistance（mΩ）
- cycle_life（次）
- self_discharge_rate（%）
- operating_temperature_min/max（°C）
- separator_thickness（mm）
- porosity（%）
- dimensions_length/width/height（附单位）

## 示例（节选）

- `technical_process`："A->B->C"（如 CN-116438709-A）— 切分为 3 个有序步骤。
- 带数值与单位：`nominal_voltage: 4.5 V` + `actions` 说明。
- 标志型字段：`value: "1"` 表示关注/具备；数值仍可能为 0，应标注未知量纲但 `has_value: true`。
- 类别含 `"null"`：`electrolyte_type: "null"` 或 `electrochemical_battery_type: ["null"]`。

## 需重点防护的边界情形

- `technical_parameters` 存在但全部为 0 + 空单位。
- `dimensions`、`separator.thickness` 单位混用。
- `actions` 文本包含数字但非结构化，避免误读为数值。

## 对宪章（v0.2.0）的影响

- 强化“两通道输入契约”与鲁棒解析规则（0/空/`"null"`→unknown，标志转布尔）。
- 保留 `actions` 与 `text_raw` 以增强可追溯性。
- 优先使用 `technical_process` 进行步骤抽取；缺失时加注记。

## 后续工作

- 扩大抽样并统计字段覆盖率。
- 起草 `technical_parameters` 与 `technical_process` 的 JSON Schema 草案。
- 实现最小单位归一工具与相应测试。
