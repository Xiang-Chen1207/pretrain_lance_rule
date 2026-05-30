# Pretraining Validation Checklist

This is a compact excerpt from `validation/validation-checklist.md` for
pretraining dataset review.

## File And Storage Checks

- [ ] `metadata.toml` exists and is valid TOML.
- [ ] `versions.toml` exists or the dataset is explicitly marked as not version-tracked.
- [ ] `description.toml` is optional and is not required for parsing, validation, restoration, sampling, or training.
- [ ] A default/single-table pretraining dataset has `dataset.lance/`.
- [ ] A partitioned pretraining dataset may omit `dataset.lance/` only when it declares at least one `[[pretrain.tables]]` entry with `role = "signal"`.
- [ ] For partitioned pretraining datasets, `storage.table_name` and `storage.lance_path` are both `__partitioned__`.
- [ ] Every declared signal table path resolves and can be opened as a Lance dataset.
- [ ] The union of declared signal tables contains at least one row.
- [ ] The sum of declared signal table `n_samples` values equals `dataset.n_samples`.

## Required Metadata Checks

- [ ] `[dataset]`, `[storage]`, `[schema]`, `[data]`, `[quality_control]`, and the modality-specific section exist.
- [ ] `dataset.task_type = "pretraining"`.
- [ ] `[pretrain]` exists.
- [ ] `pretrain.pretrain_type` is one of `self_supervised`, `supervised`, `multi_objective`, or `contrastive`.
- [ ] `pretrain.sample_id_scope = "global"` for released datasets.
- [ ] `pretrain.source_dataset_column` names an existing physical signal-table column.
- [ ] `pretrain.recording_id_column` names an existing physical signal-table column.
- [ ] `pretrain.preprocess_version` exists and is treated as the default or primary preprocessing version.
- [ ] `pretrain.split_policy` is one of `subject_disjoint`, `recording_disjoint`, `source_dataset_disjoint`, or `custom_declared`.
- [ ] If `pretrain.split_policy = "custom_declared"`, `pretrain.split_declaration_path` exists and resolves to a split policy file.

## Signal Table Checks

- [ ] Every required base column exists physically in each released signal table: `sample_id`, `subject_id`, `modality`, `data`, `shape`, `original_shape`, `valid_length`, and `qc_pass`.
- [ ] Every required pretraining column exists physically in each released signal table: `source_dataset_id`, `recording_id`, `source_path`, `start_time`, `duration`, `split`, and `preprocess_version`.
- [ ] `sample_id` values are non-empty stable strings.
- [ ] `sample_id` values are globally unique across all declared signal tables.
- [ ] Row-level `preprocess_version` is authoritative when it differs from `[pretrain].preprocess_version`.
- [ ] If `[pretrain.sampling].weight_column` is declared, the named column exists in every signal table used by the sampler.

## Source Checks

- [ ] At least one `[[pretrain.sources]]` entry exists.
- [ ] Each `[[pretrain.sources]]` entry has a unique `source_dataset_id`.
- [ ] Each source declares `source_family`, `version`, `license`, `access_level`, and `n_samples`.
- [ ] Each source `access_level` is one of `open`, `registered`, `restricted`, `internal`, or `unknown`.
- [ ] All declared `[[pretrain.sources]]` IDs appear in rows or are explicitly marked unused.

## Table And Feature Checks

- [ ] Every declared `[[pretrain.tables]]` entry has `table_id`, `role`, `lance_path`, and `n_samples`.
- [ ] Every declared table path resolves and row count matches `n_samples`.
- [ ] Feature tables declare `view_id`, `aligned_to`, and `alignment`.
- [ ] Feature alignment is `sample_id` or explicit `row_index`.
- [ ] Feature tables using `alignment = "sample_id"` contain `sample_id`.
- [ ] Feature tables using `alignment = "row_index"` contain `row_index`.
- [ ] Pure implicit row-order alignment is not used for released datasets.
- [ ] Row-level `feature_version` is authoritative when it differs from the feature-view-level default.

## EEG Channel Profile Checks

- [ ] EEG pretraining rows contain a non-empty physical `channel_profile` column.
- [ ] Non-EEG rows in multimodal pretraining datasets have `channel_profile` set to null or an empty string.
- [ ] Every non-empty `channel_profile` value is declared in `[[pretrain.channel_profiles]]`.
- [ ] Every declared `[[pretrain.channel_profiles]]` layout is one of `fixed_slot`, `variable`, or `named_only`.
- [ ] Validators do not infer channel counts from physical table names.
- [ ] Restored EEG channel-axis shape is compatible with the declared channel profile.

## Split And Sampling Checks

- [ ] Split values are stable and reproducible.
- [ ] For `subject_disjoint`, split isolation uses `subject_uid` when present, otherwise `(source_dataset_id, subject_id)`.
- [ ] For `recording_disjoint`, split isolation uses `recording_uid` when present, otherwise `(source_dataset_id, recording_id)`.
- [ ] For EEG, windows from the same recording do not appear in multiple splits unless a non-leakage exception is explicitly declared.
- [ ] Sampling policy is declared when source distributions are intentionally reweighted.
