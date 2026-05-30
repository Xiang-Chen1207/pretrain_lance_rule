# Pretrain Dataset Rule 方案

## 1. 目标

为 EEG 预训练数据集制定一套与现有 downstream dataset rule 统一的规则。预训练数据可能来自 TUH、多个 OpenNeuro dataset 或后续新增 source，因此规则需要支持多源、分区存储、无标签自监督训练、派生特征视图和可复现采样。

核心目标：

- 继续以 `metadata.toml` 作为机器可读 source of truth；
- 继续复用基础数据契约和基础必需列；
- 用 `dataset.task_type = "pretraining"` 激活 pretrain 扩展，而不是 fork 出另一套规范；
- 允许单表布局和分区布局共存；
- 对 validator、reader、sampler、trainer 都保持可落地、可校验、可复现。

## 2. 与 downstream 规则保持统一的部分

Pretrain 数据集仍然复用以下基础约定：

- 数据集目录以 `dataset_id` 命名；
- `metadata.toml` 是解析、恢复、校验、采样和训练的机器可读入口；
- `versions.toml` 记录版本变化；
- `description.toml` 只做人类可读说明，不参与机器解析；
- 基础 signal 表仍包含 `sample_id`、`subject_id`、`modality`、`data`、`shape`、`original_shape`、`valid_length`、`qc_pass`；
- EEG 仍通过 `[eeg]` 描述 sampling rate、unit、reference、montage、axis、channel layout、channel names 和 mask。

## 3. 与 downstream 不同的扩展部分

Pretrain 数据集新增 `[pretrain]` 扩展段，用来声明下游单任务数据集通常不需要的信息：

- 多 source dataset 来源，例如 `tuh`、`ds004395`；
- 全局唯一稳定的 `sample_id`；
- source 级别的 `source_dataset_id`、`recording_id`、`source_path`；
- window 级别的 `start_time`、`duration`、`split`、`preprocess_version`；
- EEG channel profile，用于 TUH 与 OpenNeuro 等不同通道空间；
- 分区 Lance 表清单 `[[pretrain.tables]]`；
- 派生 feature view，例如 NPD feature；
- 采样策略，例如 `balanced_by_source` 和 temperature reweighting。

## 4. 最终关键设计决策

### 4.1 分区布局使用明确哨兵

单表 pretrain 可以继续使用：

```text
dataset.lance/
metadata.toml
versions.toml
description.toml
```

分区 pretrain 不强制存在单一 `dataset.lance/`。此时 `[storage]` 必须写成：

```toml
[storage]
table_name = "__partitioned__"
lance_path = "__partitioned__"
```

`__partitioned__` 表示 reader 和 validator 必须从 `[[pretrain.tables]]` 中所有 `role = "signal"` 的表解析逻辑 signal table。

### 4.2 必需字段必须是物理列

为了让 validator 可以稳定落地，规则不再使用 "contain or expose" 这类虚拟 reader 语义。每个发布的 signal 表必须包含物理列：

```text
source_dataset_id
recording_id
source_path
start_time
duration
split
preprocess_version
```

EEG 行还必须包含非空且已声明的 `channel_profile`。多模态预训练中的非 EEG 行可以把 `channel_profile` 置为 null 或空字符串。

### 4.3 行级版本是 ground truth

`[pretrain].preprocess_version` 是默认或主要 pipeline 版本。每行的 `preprocess_version` 列才是精确 ground truth。

Feature view 同理：`[[pretrain.feature_views]].feature_version` 是默认值，feature 表中的行级 `feature_version` 优先级更高。

### 4.4 Feature 对齐不使用隐式 row order

Feature table 只能用两种方式对齐 signal table：

- `alignment = "sample_id"`：feature 表包含同一批 `sample_id`；
- `alignment = "row_index"`：feature 表包含显式 `row_index` 外键，引用 `aligned_to` signal 表行索引。

发布数据集不得使用纯隐式 row-order 对齐，避免静默错位。

### 4.5 Split policy 是闭集

`split_policy` 必须是以下值之一：

```text
subject_disjoint
recording_disjoint
source_dataset_disjoint
custom_declared
```

如果使用 `custom_declared`，必须提供 `split_declaration_path`，并让该文件描述 split 隔离规则。

`subject_disjoint` 使用 `subject_uid`，没有时使用 `(source_dataset_id, subject_id)`。`recording_disjoint` 使用 `recording_uid`，没有时使用 `(source_dataset_id, recording_id)`。

## 5. 推荐落地目录

单表布局：

```text
<dataset_id>/
  dataset.lance/
  metadata.toml
  versions.toml
  description.toml
  derivatives/
    features/
      <feature_view_id>.lance/
```

分区布局：

```text
<dataset_id>/
  metadata.toml
  versions.toml
  description.toml
  tables/
    signal/
      tuh_train.lance/
      openneuro_ds004395_slot136_train.lance/
    features/
      npd_v1/
        openneuro_ds004395_slot136_train.lance/
```

## 6. 新增 pretrain dataset 的流程

1. 确定 source dataset 列表和 source ID 命名，例如 `tuh`、`ds004395`。
2. 为每个样本生成跨所有物理表全局唯一的 `sample_id`。
3. 写入基础必需列和 pretrain 必需物理列。
4. 根据 EEG 通道空间定义 `[[pretrain.channel_profiles]]`。
5. 如果数据量或通道 profile 需要分区，声明所有 `[[pretrain.tables]]`，并把 `[storage]` 写成 `__partitioned__`。
6. 如果存在 NPD 或其他派生特征，声明 `[[pretrain.feature_views]]` 和对应 feature table。
7. 声明 split policy，并验证 subject/recording/source 级别无泄漏。
8. 声明 sampling policy，尤其是行分布不等于训练采样概率时。
9. 按 `validation-checklist-pretrain.md` 做本地 blocking validation。

## 7. 仍需实现的配套工作

- 在实际 validator 中实现分区表打开、row count 汇总、全局 `sample_id` 去重；
- 在 reader 中把 `__partitioned__` 解析为多个 signal tables 的逻辑 union；
- 在 sampler 中读取 `[pretrain.sampling]`，支持 source balance 和 `sample_weight`；
- 为 feature view 实现 `sample_id` 或显式 `row_index` 对齐检查；
- 针对大规模数据，把全局唯一性校验实现为可外排或分片的算法。
