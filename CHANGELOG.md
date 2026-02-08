# Changelog

All notable changes to the GPU Instance Specification will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2026-02-08

### Added

- Initial GIS v1.0.0 specification (`spec.md`)
- JSON Schema (draft-07) at `schema/v1.json`
- 6 top-level sections: `gpu`, `compute`, `pricing`, `availability`, `normalized`, `meta`
- 5 example GIS documents:
  - AWS P5 (8×H100 SXM5)
  - CoreWeave H100 SXM
  - Lambda Labs H100 SXM5
  - RunPod A100 80GB
  - Vast.ai A100 80GB
- Normalization algorithm reference (`NORMALIZATION.md`)
  - `cost_per_gpu_hour` — per-GPU hourly cost
  - `cost_per_tflop_hour` — price-performance metric
  - `vram_per_dollar` — memory value metric
- Contributing guide (`CONTRIBUTING.md`)
- Human-readable specification (`spec.md`)
- CC BY 4.0 license
