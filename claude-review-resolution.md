# Claude CLI Review Resolution

The external CLI review gave a conditional pass: the extension direction is
reasonable, but several ambiguities needed to be fixed before sharing the rule.

## Resolved Blocking Findings

| Finding | Resolution |
|---|---|
| C1: "contain or expose" was not validator-safe | Removed the virtual/exposed-column wording. Required pretraining fields are now physical columns in released signal tables. |
| C2: partitioned layout conflicted with `storage.lance_path = "dataset.lance"` | Partitioned pretraining datasets now use `table_name = "__partitioned__"` and `lance_path = "__partitioned__"`. Readers resolve signal tables from `[[pretrain.tables]]`. |
| C3: metadata-level and row-level versions could conflict | `[pretrain].preprocess_version` and feature-view `feature_version` are defaults. Row-level values are authoritative. |
| C4: implicit `row_order` alignment was fragile | Removed implicit row-order alignment. Feature tables align by `sample_id` or explicit `row_index`. |
| C5: `split_policy` was not a closed enum | `split_policy` is now one of `subject_disjoint`, `recording_disjoint`, `source_dataset_disjoint`, or `custom_declared`; custom policies require `split_declaration_path`. |

## Resolved High And Medium Findings

| Finding | Resolution |
|---|---|
| `subject_id` and `recording_id` are source-scoped | Split isolation now uses `subject_uid` / `recording_uid` when present, otherwise compound keys with `source_dataset_id`. |
| Non-EEG `channel_profile` behavior was undefined | Non-EEG rows must use null or empty string; validators only require declared non-empty profiles for EEG rows. |
| Channel profile layout lacked enum | Added `fixed_slot`, `variable`, and `named_only`. |
| `access_level` lacked enum | Added `open`, `registered`, `restricted`, `internal`, and `unknown`. |
| `source_family` needed stable naming guidance | Added stable lowercase identifier guidance while allowing future families without spec changes. |
| `[pretrain].notes` duplicated `description.toml` | Removed recommended `notes` from `[pretrain]`. |
| `registry.json` was an implementation detail | Reframed generated registry/index files as caches, not required spec files unless declared as `role = "index"`. |
| `pretrain_type = "multitask"` was ambiguous | Replaced it with `multi_objective`. |
| Partitioned dataset without `dataset.lance/` needed explicit failure behavior | Validation now requires at least one declared signal table when `dataset.lance/` is absent. |
| Sampling `weight_column` could reference a missing field | Validation now requires the named weight column to exist in every signal table used by the sampler. |

## Remaining Implementation Notes

- The specification requires exact global `sample_id` uniqueness. Large datasets should implement this with a scalable validator, such as hash partitioning or an external-sort pass.
- The rule allows new `source_family` values without changing the spec. Platform validators may still maintain local registries for known families.
- The preview folder is for review convenience; canonical source of truth remains the normal repo files under `rules/`, `schemas/`, `examples/`, and `validation/`.
