<p align="center">
  <img src="https://computespec.dev/favicon.svg" width="80" height="80" alt="computespec logo" />
</p>

<h1 align="center">GPU Instance Specification (GIS)</h1>

<p align="center">
  <strong>The open standard for describing GPU cloud offerings.</strong><br />
  One JSON document. Any GPU. Any provider. Fully described.
</p>

<p align="center">
  <a href="https://creativecommons.org/licenses/by/4.0/"><img src="https://img.shields.io/badge/license-CC%20BY%204.0-10b981?style=flat-square" alt="License: CC BY 4.0" /></a>
  <a href="https://computespec.dev/gpu/v1/schema.json"><img src="https://img.shields.io/badge/spec-v1.0.0-10b981?style=flat-square" alt="Spec v1.0.0" /></a>
  <a href="https://computespec.dev"><img src="https://img.shields.io/badge/site-computespec.dev-0ea5e9?style=flat-square" alt="computespec.dev" /></a>
  <a href="https://github.com/nxgeninfra/gis-spec/issues"><img src="https://img.shields.io/github/issues/nxgeninfra/gis-spec?style=flat-square&color=64748b" alt="Issues" /></a>
</p>

<p align="center">
  <a href="#quick-start">Quick Start</a> ·
  <a href="./spec.md">Full Spec</a> ·
  <a href="./schema/v1.json">JSON Schema</a> ·
  <a href="./examples/">Examples</a> ·
  <a href="https://computespec.dev">Website</a>
</p>

---

## The Problem

The GPU cloud market has **50+ providers** and **$50B+ in annual spend** — but zero common format for describing offerings.

```
AWS calls it          p5.48xlarge
Lambda Labs calls it  gpu_1x_h100_sxm5
RunPod calls it       a100-80gb-sxm
CoreWeave calls it    h100-sxm-80gb

Same hardware. No shared vocabulary.
```

Every comparison tool, procurement system, and AI agent independently scrapes, normalizes, and structures the same data — producing **incompatible representations** that cannot be composed, validated, or versioned.

**GIS fixes that.** One JSON schema. One document per instance. Machine-readable. Vendor-neutral.

---

## Quick Start

A GIS document fully describes one GPU cloud offering:

```json
{
  "$schema": "https://computespec.dev/gpu/v1/schema.json",
  "provider": "lambda-labs",
  "instance": "gpu_1x_h100_sxm5",
  "gpu": {
    "model": "nvidia-h100",
    "variant": "sxm5",
    "count": 1,
    "vram_gb": 80,
    "tflops_fp16": 989.5,
    "memory_bandwidth_tbps": 3.35,
    "interconnect": "nvlink",
    "architecture": "hopper"
  },
  "compute": {
    "vcpus": 26,
    "ram_gb": 200,
    "storage_gb": 512,
    "storage_type": "nvme-ssd",
    "network_gbps": 25
  },
  "pricing": {
    "currency": "USD",
    "billing_unit": "per-hour",
    "billing_granularity": "per-second",
    "on_demand": 2.49,
    "spot": null,
    "reserved_1yr": null,
    "reserved_3yr": null
  },
  "normalized": {
    "cost_per_gpu_hour": 2.49,
    "cost_per_tflop_hour": 0.00252,
    "vram_per_dollar": 32.13
  },
  "meta": {
    "last_updated": "2026-02-08T00:00:00Z",
    "source_url": "https://lambdalabs.com/service/gpu-cloud",
    "verified": true,
    "spec_version": "1.0.0"
  }
}
```

### Validate

```bash
# Using ajv-cli
npx ajv validate -s schema/v1.json -d examples/lambda-h100.json

# Using Python jsonschema
python -m jsonschema -i examples/lambda-h100.json schema/v1.json
```

---

## Spec Structure

A GIS document has **6 sections**. 4 required, 2 optional. Every field earns its place.

| Section | Required | Purpose | Key Fields |
|---------|----------|---------|------------|
| **`gpu`** | ✅ | GPU hardware | `model`, `variant`, `count`, `vram_gb`, `tflops_fp16`, `memory_bandwidth_tbps`, `interconnect`, `architecture` |
| **`compute`** | ✅ | CPU, RAM, storage, network | `vcpus`, `ram_gb`, `storage_gb`, `storage_type`, `network_gbps` |
| **`pricing`** | ✅ | Cost structure | `currency`, `billing_unit`, `billing_granularity`, `on_demand`, `spot`, `reserved_1yr`, `reserved_3yr` |
| **`meta`** | ✅ | Provenance | `last_updated`, `source_url`, `verified`, `spec_version` |
| **`availability`** | ○ | Regions and SLA | `regions`, `type`, `sla_uptime` |
| **`normalized`** | ○ | Derived comparison metrics | `cost_per_gpu_hour`, `cost_per_tflop_hour`, `vram_per_dollar` |

> **Extensibility:** Additional fields are allowed at any level. Validators MUST ignore unknown fields. Tools SHOULD preserve them.

---

## Normalization Algorithms

The `normalized` section contains **computed metrics** — derived from `pricing` and `gpu` fields, never provider-reported. This ensures apples-to-apples comparison.

### cost_per_gpu_hour

Effective cost per individual GPU per hour, normalized to USD hourly.

```
cost_per_gpu_hour = (on_demand / gpu.count) × unit_multiplier

where unit_multiplier:
  per-hour   → 1
  per-minute → 60
  per-second → 3600
  per-month  → 1/730
```

### cost_per_tflop_hour

Cost per FP16 TFLOP of compute per hour.

```
cost_per_tflop_hour = cost_per_gpu_hour / gpu.tflops_fp16
```

Returns `null` if `tflops_fp16` is null.

### vram_per_dollar

GB of VRAM you get per dollar per hour.

```
vram_per_dollar = gpu.vram_gb / cost_per_gpu_hour
```

---

## Design Principles

| Principle | Rule |
|-----------|------|
| **Minimal** | Only fields that 80%+ of providers can populate |
| **Flat** | Avoid deep nesting — max 2 levels |
| **Nullable** | Fields not all providers support MUST be nullable |
| **Computed** | `normalized` metrics are derived, never provider-reported |
| **Versioned** | `spec_version` in every document, schema URL includes version |
| **Extensible** | Unknown fields preserved, never rejected |
| **Neutral** | Describes offerings — never recommends providers |

---

## Examples

| File | Provider | GPU | $/GPU/hr |
|------|----------|-----|----------|
| [`lambda-h100.json`](./examples/lambda-h100.json) | Lambda Labs | 1× H100 SXM5 | $2.49 |
| [`aws-p5-48xlarge.json`](./examples/aws-p5-48xlarge.json) | AWS | 8× H100 SXM5 | $12.29 |
| [`runpod-a100.json`](./examples/runpod-a100.json) | RunPod | 1× A100 80GB | $1.64 |
| [`coreweave-h100.json`](./examples/coreweave-h100.json) | CoreWeave | 1× H100 SXM | $2.23 |
| [`vastai-a100.json`](./examples/vastai-a100.json) | Vast.ai | 1× A100 80GB | $1.10 |

---

## Repository Structure

```
gis-spec/
├── spec.md                       # Human-readable specification
├── schema/
│   └── v1.json                   # JSON Schema (draft-07)
├── examples/
│   ├── aws-p5-48xlarge.json      # AWS P5 (8×H100)
│   ├── coreweave-h100.json       # CoreWeave H100
│   ├── lambda-h100.json          # Lambda Labs H100
│   ├── runpod-a100.json          # RunPod A100
│   └── vastai-a100.json          # Vast.ai A100
├── NORMALIZATION.md              # Normalization algorithm reference
├── CHANGELOG.md                  # Version history
├── CONTRIBUTING.md               # How to contribute
├── LICENSE                       # CC BY 4.0
└── README.md                     # This file
```

---

## For AI Agents

GIS documents are designed to be consumed by LLMs, AI agents, and automated systems:

- **Structured JSON** — no scraping or HTML parsing required
- **Consistent schema** — same fields, same types, every provider
- **Self-describing** — `$schema` field points to the JSON Schema for validation
- **Normalized metrics** — pre-computed comparison values, no math needed
- **Versioned** — `meta.spec_version` tells you exactly which spec version to expect

To use GIS data in your agent or tool, validate against the schema and read the `normalized` section for instant comparison.

```python
import json

with open("lambda-h100.json") as f:
    gis = json.load(f)

provider = gis["provider"]                          # "lambda-labs"
gpu_model = gis["gpu"]["model"]                     # "nvidia-h100"
price = gis["normalized"]["cost_per_gpu_hour"]       # 2.49
efficiency = gis["normalized"]["cost_per_tflop_hour"] # 0.00252
```

---

## Why a Specification, Not a Product

Every player in the GPU cloud space builds a **product on top of unstructured data**. None builds the **format the data should be in**.

| Approach | Examples | Limitation |
|----------|----------|------------|
| Proprietary indices | Silicon Data ($4.7M), Ornn ($5.7M) | Closed data, financial audience |
| Marketplaces | Shadeform (YC S23) | Transaction layer, not a standard |
| Comparison sites | getdeploying.com | Proprietary normalization, no schema |
| Open inventories | Spare Cores (EU grant) | Product, not a specification |
| **Open specification** | **GIS** | **The layer underneath all of them** |

Think **OpenAPI for GPU cloud**. The spec is free forever. If a competitor adopts GIS format — we won.

---

## Roadmap

GIS is the first specification in the **computespec** family:

| Spec | Scope | Status |
|------|-------|--------|
| **GIS** — GPU Instance Specification | GPU cloud offerings | **v1.0.0 — Live** |
| **CIS** — CPU Instance Specification | CPU cloud offerings | Planned (500+ GIS stars) |
| **SPS** — Spot Pricing Specification | Spot/preemptible pricing | Planned |
| **SLA-S** — SLA Specification | Service level agreements | Planned |

---

## Contributing

We welcome contributions — new provider examples, spec feedback, schema improvements, documentation.

- **Add a provider:** Submit a GIS document for a provider not yet covered
- **Spec feedback:** Open an issue or email [spec@computespec.dev](mailto:spec@computespec.dev)
- **Fix data:** If pricing or specs are wrong, submit a PR with the corrected values and source URL

See [CONTRIBUTING.md](./CONTRIBUTING.md) for details.

---

## License

This specification is licensed under [**Creative Commons Attribution 4.0 International (CC BY 4.0)**](https://creativecommons.org/licenses/by/4.0/).

You are free to share and adapt for any purpose, including commercial use, as long as you give appropriate credit.

---

<p align="center">
  <a href="https://computespec.dev"><strong>computespec.dev</strong></a> · 
  <a href="https://github.com/nxgeninfra">GitHub</a> · 
  <a href="https://twitter.com/nxgeninfra">Twitter</a> · 
  <a href="mailto:spec@computespec.dev">spec@computespec.dev</a>
</p>

<p align="center">
  <em>"GPUs are boring when you can describe them in one JSON document."</em>
</p>
