# GPU Instance Specification (GIS)

**Version:** 1.0.0
**Status:** Draft
**License:** CC BY 4.0
**Schema:** https://computespec.dev/gpu/v1/schema.json

---

## 1. Introduction

The GPU Instance Specification (GIS) defines a vendor-neutral, machine-readable JSON format for describing GPU cloud offerings. A single GIS document fully describes one GPU instance from one provider — including hardware, compute resources, pricing, availability, and normalized comparison metrics.

### 1.1 Goals

- Provide a common language for describing GPU cloud offerings
- Enable apples-to-apples comparison across providers
- Be machine-readable (JSON) and human-readable (clear field names)
- Be minimal — only fields that 80%+ of providers can populate
- Be extensible — additional fields are allowed and preserved

### 1.2 Non-Goals

- GIS does not recommend providers
- GIS does not define benchmarks or performance tests
- GIS does not handle multi-GPU cluster topologies (future spec)
- GIS does not define API endpoints or query formats

---

## 2. Document Structure

A GIS document is a JSON object with the following top-level fields:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `$schema` | string | No | Reference to the GIS JSON Schema URL |
| `provider` | string | Yes | Provider identifier (kebab-case) |
| `instance` | string | Yes | Provider's instance type identifier |
| `gpu` | object | Yes | GPU hardware specification |
| `compute` | object | Yes | Compute bundle (CPU, RAM, storage, network) |
| `pricing` | object | Yes | Pricing information |
| `availability` | object | No | Availability information |
| `normalized` | object | No | Derived comparison metrics |
| `meta` | object | Yes | Document metadata |

---

## 3. Field Definitions

### 3.1 `provider` (required)

Provider identifier in kebab-case. Examples: `aws`, `gcp`, `lambda-labs`, `runpod`, `coreweave`.

### 3.2 `instance` (required)

The provider's own instance type identifier. Examples: `p5.48xlarge`, `gpu_1x_h100_sxm5`, `a100-80gb-sxm`.

### 3.3 `gpu` (required)

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `model` | string | Yes | GPU model (e.g., `nvidia-h100`, `nvidia-a100`) |
| `variant` | string \| null | No | GPU variant (e.g., `sxm5`, `pcie`, `80gb`) |
| `count` | integer | Yes | Number of GPUs (≥ 1) |
| `vram_gb` | number | Yes | VRAM per GPU in GB |
| `tflops_fp16` | number \| null | No | Peak FP16 TFLOPS per GPU |
| `memory_bandwidth_tbps` | number \| null | No | Memory bandwidth in TB/s per GPU |
| `interconnect` | string \| null | No | Interconnect type (`nvlink`, `pcie`, `nvswitch`) |
| `architecture` | string \| null | No | Architecture generation (`hopper`, `ampere`, `ada-lovelace`) |

### 3.4 `compute` (required)

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `vcpus` | integer | Yes | Number of virtual CPUs |
| `ram_gb` | number | Yes | System RAM in GB |
| `storage_gb` | number \| null | No | Local storage in GB |
| `storage_type` | string \| null | No | Storage type (`nvme-ssd`, `ssd`, `hdd`) |
| `network_gbps` | number \| null | No | Network bandwidth in Gbps |

### 3.5 `pricing` (required)

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `currency` | string | Yes | ISO 4217 currency code |
| `billing_unit` | string | Yes | `per-hour`, `per-minute`, `per-second`, `per-month` |
| `billing_granularity` | string \| null | No | Minimum billing increment |
| `on_demand` | number \| null | No | On-demand price per billing_unit |
| `spot` | number \| null | No | Spot/preemptible price per billing_unit |
| `reserved_1yr` | number \| null | No | 1-year reserved price per billing_unit |
| `reserved_3yr` | number \| null | No | 3-year reserved price per billing_unit |

### 3.6 `availability` (optional)

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `regions` | string[] | No | Available regions |
| `type` | string \| null | No | Availability type (`on-demand`, `spot`, `reserved`) |
| `sla_uptime` | number \| null | No | SLA uptime as decimal (0.999 = 99.9%) |

### 3.7 `normalized` (optional)

Derived comparison metrics. These MUST be computed from the `pricing` and `gpu` fields using the algorithms defined in [NORMALIZATION.md](./NORMALIZATION.md). They are never provider-reported.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `cost_per_gpu_hour` | number \| null | No | Effective cost per GPU per hour (USD) |
| `cost_per_tflop_hour` | number \| null | No | Cost per FP16 TFLOP per hour (USD) |
| `vram_per_dollar` | number \| null | No | GB of VRAM per dollar per hour |

### 3.8 `meta` (required)

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `last_updated` | string (ISO 8601) | Yes | Timestamp of last data update |
| `source_url` | string (URI) \| null | No | Provider's pricing page URL |
| `verified` | boolean | No | Whether data has been manually verified |
| `spec_version` | string (semver) | Yes | GIS spec version |

---

## 4. Extensibility

GIS documents MAY contain additional fields at any level. Validators MUST ignore unknown fields. Tools SHOULD preserve unknown fields when processing GIS documents.

---

## 5. Versioning

GIS follows Semantic Versioning:
- **Major:** Breaking changes (field removals, type changes)
- **Minor:** New optional fields, new enum values
- **Patch:** Clarifications, typo fixes

The `meta.spec_version` field MUST contain the spec version the document conforms to.

---

## 6. Naming Conventions

- Provider identifiers: kebab-case (`lambda-labs`, not `Lambda Labs`)
- GPU models: lowercase kebab-case (`nvidia-h100`, not `NVIDIA H100`)
- Architecture: lowercase kebab-case (`ada-lovelace`, not `Ada Lovelace`)
- All units are explicit in field names (`_gb`, `_tbps`, `_gbps`)

---

*Published by [nxgeninfra](https://github.com/nxgeninfra) under CC BY 4.0.*
