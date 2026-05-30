# Pretrain Dataset Rule Preview

This folder is a compact review bundle for the pretraining dataset rule.

Canonical source files still live in the repository's normal locations. Files
here are copied or summarized so they can be shared as one self-contained
package.

## Contents

| File | Purpose |
|---|---|
| `pretrain-rule-plan.cn.md` | Chinese plan and rationale for the pretraining rule |
| `pretrain-dataset-rules.md` | English normative rule text |
| `pretrain-dataset-rules.cn.md` | Chinese normative rule text |
| `pretrain-metadata.template.toml` | Empty pretraining metadata template |
| `pretrain-eeg.metadata.example.toml` | EEG pretraining example based on TUH plus OpenNeuro-style sources |
| `pretrain-eeg.versions.example.toml` | Version history example for a partitioned pretraining dataset |
| `pretrain-eeg.description.example.toml` | Human-readable description example for a pretraining dataset |
| `dataset-tree.txt` | Minimal dataset directory examples |
| `validation-checklist-pretrain.md` | Pretraining-specific validation excerpt |
| `claude-review-resolution.md` | Summary of external CLI review findings and how they were resolved |

## Canonical Locations

| Preview file | Canonical source |
|---|---|
| `pretrain-dataset-rules.md` | `rules/pretrain-dataset-rules.md` |
| `pretrain-dataset-rules.cn.md` | `rules/pretrain-dataset-rules.cn.md` |
| `pretrain-metadata.template.toml` | `schemas/pretrain-metadata.template.toml` |
| `pretrain-eeg.metadata.example.toml` | `examples/pretrain-eeg/metadata.toml` |
| `pretrain-eeg.versions.example.toml` | `examples/pretrain-eeg/versions.toml` |
| `pretrain-eeg.description.example.toml` | `examples/pretrain-eeg/description.toml` |
| `dataset-tree.txt` | `examples/dataset-tree.txt` |
| `validation-checklist-pretrain.md` | summarized from `validation/validation-checklist.md` |

## Review Notes

The current version incorporates the external CLI review suggestions:

- partitioned pretraining datasets use the `__partitioned__` storage sentinel;
- required pretraining fields are physical Lance columns, not virtual reader fields;
- row-level `preprocess_version` and `feature_version` are authoritative;
- feature alignment uses `sample_id` or explicit `row_index`, not implicit row order;
- `split_policy`, `access_level`, and channel profile layout semantics are explicit.
