# GIS Normalization Algorithms

**Version:** 1.0.0
**Status:** Normative
**License:** CC BY 4.0

---

## Overview

The `normalized` section of a GIS document contains **derived comparison metrics**. These values are computed from the `pricing` and `gpu` sections — they are never provider-reported.

This ensures apples-to-apples comparison across providers with different billing models, GPU counts, and pricing structures.

All normalized values are denominated in **USD per hour** regardless of the provider's native billing unit or currency.

---

## Unit Multiplier

All normalization begins by converting the provider's billing unit to an hourly rate.

```
unit_multiplier(billing_unit) =
  per-second → 3600
  per-minute → 60
  per-hour   → 1
  per-month  → 1/730
```

The per-month divisor of 730 represents the average hours per month (365.25 days × 24 hours / 12 months).

---

## Algorithm 1: cost_per_gpu_hour

**Purpose:** Effective cost per individual GPU per hour.

**Formula:**

```
cost_per_gpu_hour = (pricing.on_demand × unit_multiplier) / gpu.count
```

**Rules:**
- Returns `null` if `pricing.on_demand` is null
- Uses `on_demand` price (not spot or reserved)
- Divides by `gpu.count` to get per-GPU cost for multi-GPU instances
- Result is in USD regardless of `pricing.currency` (currency conversion is out of scope for v1.0)

**Example — Lambda Labs H100:**

```
on_demand = 2.49, billing_unit = "per-hour", gpu.count = 1
cost_per_gpu_hour = (2.49 × 1) / 1 = 2.49
```

**Example — AWS P5 (8×H100):**

```
on_demand = 98.32, billing_unit = "per-hour", gpu.count = 8
cost_per_gpu_hour = (98.32 × 1) / 8 = 12.29
```

---

## Algorithm 2: cost_per_tflop_hour

**Purpose:** Cost per FP16 TFLOP of compute per hour. Measures price-performance.

**Formula:**

```
cost_per_tflop_hour = cost_per_gpu_hour / gpu.tflops_fp16
```

**Rules:**
- Returns `null` if `cost_per_gpu_hour` is null
- Returns `null` if `gpu.tflops_fp16` is null
- Uses peak theoretical FP16 TFLOPS (not measured benchmarks)
- Lower is better

**Example — Lambda Labs H100:**

```
cost_per_gpu_hour = 2.49, tflops_fp16 = 989.5
cost_per_tflop_hour = 2.49 / 989.5 = 0.00252
```

---

## Algorithm 3: vram_per_dollar

**Purpose:** GB of VRAM per dollar per hour. Measures memory value.

**Formula:**

```
vram_per_dollar = gpu.vram_gb / cost_per_gpu_hour
```

**Rules:**
- Returns `null` if `cost_per_gpu_hour` is null or zero
- Uses per-GPU VRAM (not total across all GPUs)
- Higher is better

**Example — Vast.ai A100:**

```
vram_gb = 80, cost_per_gpu_hour = 1.10
vram_per_dollar = 80 / 1.10 = 72.73
```

---

## Precision

- All normalized values SHOULD be rounded to a reasonable precision
- `cost_per_gpu_hour`: 2 decimal places
- `cost_per_tflop_hour`: 5 decimal places
- `vram_per_dollar`: 2 decimal places

---

## Future Metrics

The following metrics are under consideration for future spec versions:

| Metric | Formula | Purpose |
|--------|---------|---------|
| `cost_per_vram_gb_hour` | `cost_per_gpu_hour / gpu.vram_gb` | Cost per GB of VRAM per hour |
| `bandwidth_per_dollar` | `gpu.memory_bandwidth_tbps / cost_per_gpu_hour` | Memory bandwidth value |
| `spot_discount_pct` | `1 - (pricing.spot / pricing.on_demand)` | Spot discount percentage |

These will be added when community feedback indicates demand.

---

*Published by [nxgeninfra](https://github.com/nxgeninfra) under CC BY 4.0.*
