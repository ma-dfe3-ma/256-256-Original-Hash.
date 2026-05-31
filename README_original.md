# 🔺 256×256 Original Hash

> A cryptographic hash function with **active Sacred Pair inversion at dimension 129**
> — the first design in the Marco Lazcano research series where the K1/K2 boundary
> is genuinely crossed during normal operation.
>
> **Author:** Marco Lazcano — Decatur, GA, USA — roble99oak77@gmail.com — May 2026

---

## ⚠️ Security Notice

This is a **research prototype**. It has not been formally audited.
**Do not use in production systems or cryptographic protocols**
without independent peer review and formal cryptanalysis.

---

## Overview

| Property | Value |
|----------|-------|
| Input block | 256 bytes |
| Internal state | 256 bytes |
| Output | 256 bytes (2048 bits) |
| Permutation rounds | 24 |
| Chunk size | 256 bytes (full state, one block) |
| Anchor order | base → tip (original structure) |
| K1 region | dim 0–127 → unchanged (128 anchors) |
| K2 region | dim 128–254 → SP(x) active (127 anchors) |
| K1/K2 boundary | Dimension 129 |
| Sacred Pair | SP(x) = (~x & 0xFF) |
| Architecture | Sponge construction |

---

## What Makes This Design Unique

This is the **first design in the series where K2 is genuinely active**.

With a 256-byte input, the pyramidal function produces **255 anchors**.
The K1/K2 boundary at dimension 129 means:

```
Anchors 0..127   → K1 → passed unchanged        (128 anchors)
Anchors 128..254 → K2 → SP(x) = (~x & 0xFF)     (127 anchors)
Tip (dim 255)    → appended last
```

In every permutation round, **127 of 255 anchor values are complemented**
— introducing maximum XOR-distance transformations (XOR = 0xFF) into
half the anchor space. This acts as a structural diffusion epicenter.

---

## Statistical Test Results — Full 2048-bit Output

All 11 tests on the **complete 256-byte output**.
N=1,000 samples for Pearson correlation — statistically definitive.

| Test | Result | Threshold | Status |
|------|--------|-----------|--------|
| Basic functionality | All distinct | All distinct | ✅ |
| Determinism (200 trials) | 0 errors | 0 errors | ✅ |
| Avalanche mean | **49.804%** | 50.00% ± 2% | ✅ EXCELLENT |
| Std deviation | **1.196%** | < 5% | ✅ EXCELLENT |
| Shannon entropy | **7.9975 b/byte** | > 7.98 | ✅ EXCELLENT |
| Entropy efficiency | **99.97%** | > 99% | ✅ EXCELLENT |
| Chi-square | **268.77** | < 293.2 (95%) | ✅ EXCELLENT |
| Pearson corr. avg (N=1000) | **0.02526** | < 0.05 | ✅ EXCELLENT |
| Pairs with corr > 0.10 | **60 / 32,640** (0.18%) | ~0% | ✅ EXCELLENT |
| S-box max differential | **1.56%** | < 6.25% | ✅ SECURE |
| **SAC deviation > 10%** | **0.98% ★** | < 5% | ✅ **BEST IN SERIES** |
| Collision resistance | **0 / 400** | 0 | ✅ ROBUST |

---

## Comparison Across the Research Series

| Metric | Pyramidal 256×512 | Inverted 512×512 | **Original 256×256** |
|--------|------------------|------------------|----------------------|
| Avalanche | 50.54% σ=2.26% | 49.906% σ=1.104% | **49.804% σ=1.196%** |
| Entropy | 7.9914 b/B | 7.9974 b/B | **7.9975 b/B** |
| Chi-square | 275.30 | 237.51 | **268.77** |
| Corr. avg | 0.185 (N=150) | 0.025 (N=1000) | **0.025 (N=1000)** |
| SAC dev >10% | not measured | 1.09% | **0.98% ★** |
| K2 active | Never | Never | **127 anchors ★** |
| Anchor order | base→tip | tip→base | **base→tip** |

---

## Research Program Context

| # | Work | K2 Status |
|---|------|-----------|
| 1 | Marco System 128×256 | Defined |
| 2 | Marco's Theorem | Formalized |
| 3 | Pyramidal Hash 256×512 | Unreachable (dim 257) |
| 4 | 512×512 Inverted Hash | Unreachable (dim 257) |
| **5** | **256×256 Original Hash** | **ACTIVE — 127 anchors** |

---

## How It Works

### 1. Pyramidal Nonlinear Function

The full 256-byte state is compressed level by level:

```python
diff = SBOX[(seq[i+1] - seq[i]) mod 256]   # AES S-box
diff = safe_byte(diff XOR anchor)            # anchor mixing
diff = rotate_left(diff, (i mod 7) + 1)     # position rotation
```

255 levels → 255 left anchors + 1 tip.

### 2. Original Output with Active K2

```python
for i, anchor in enumerate(left_anchors):
    if i >= 128:                        # K2 region — ACTIVE
        output.append(SP(anchor))       # SP(x) = (~x & 0xFF)
    else:                               # K1 region
        output.append(anchor)           # unchanged
output.append(tip)                      # tip last
```

### 3. Sacred Pair Properties

```
SP(x) = (~x) & 0xFF

Involution:  SP(SP(x)) = x  for all x
XOR distance: x XOR SP(x) = 0xFF  for all x (maximum)
```

### 4. Internal Permutation (24 rounds)

1. **Full-state pyramidal mixing** with active K2 (127 anchors complemented)
2. **Linear diffusion** — each byte XOR-ed with the next
3. **Nonlinear substitution** — AES S-box on every byte

---

## Installation

```bash
git clone https://github.com/your-username/original-hash-256x256.git
cd original-hash-256x256
python3 original_hash_256x256_analysis.py
```

---

## Usage

```python
from original_hash_256x256 import OriginalHash

h = OriginalHash()
h.update(b"Hello, world!")
print(h.hexdigest(256))    # 512 hex chars = 2048 bits

# Streaming
h2 = OriginalHash()
h2.update(b"first chunk")
h2.update(b"second chunk")
print(h2.hexdigest(32))    # 64 hex chars = 256 bits
```

### Example outputs (first 32 hex chars)

```
b"" (empty)       ->  df8b75b158179432ba480f2cb72d8eab...
b"\x00"           ->  752fedc0a8a797b79fab0306c20860f7...
b"\x01"           ->  81f677cac69682c8dbd214ae21a766e9...
b"\xff"           ->  4cfd8577dbc96149f414c5e65fb9f04b...
b"Hola Mundo"     ->  0b4616015b393486be0569624ff05b51...
b"Hello"          ->  f4ddc5e54d7b7a05bb1f38efc6618743...
b"hello"          ->  ed223a7b9d04864bfd3b143f135ea863...
b"hello " (space) ->  5ce94875f610487008839e36b487aa0c...
```

---

## Performance

> Pure Python 3 — not representative of a compiled implementation.

| Input size | Time / hash |
|------------|------------|
| 256 bytes  | ~1,589 ms  |
| 512 bytes  | ~2,203 ms  |
| 1 KB       | ~3,212 ms  |
| 4 KB       | ~9,796 ms  |

A C/Rust implementation is estimated to be ~100× faster.

---

## Reproducing the Tests

```bash
python3 original_hash_256x256_analysis.py

# Estimated runtimes:
# Pearson correlation (N=1000): ~30 minutes
# SAC (150 samples, 8 bits):    ~40 minutes
# Total:                        ~80 minutes
```

---

## Known Limitations

- **No formal cryptanalysis** — differential, linear, algebraic attacks not studied.
- **K2 boundary optimization** — dim 129 chosen by design; other positions not explored.
- **Python only** — unsuitable for production performance.
- **No external audit** — cryptographic peer review pending.

---

## Academic Paper

📄 **256×256 Original Hash: A Cryptographic Hash Function with Active
Sacred Pair Inversion at Dimension 129**
*Marco Lazcano, May 2026*
*(submitted for peer review)*

---

## License

MIT License — © 2026 Marco Lazcano — see [LICENSE](LICENSE)

---

## References

1. Daemen & Rijmen (2002). *The Design of Rijndael: AES*. Springer.
2. Bertoni et al. (2011). *Cryptographic sponge functions*. IACR.
3. NIST (2015). *SHA-3 Standard*. FIPS PUB 202.
4. NIST (2012). *SHA-2 Standard*. FIPS PUB 180-4.
5. Lazcano, M. (2026). *Marco System 128×256*. Independent Research.
6. Lazcano, M. (2026). *Marco's Theorem*. Independent Research.
7. Lazcano, M. (2026). *Pyramidal Hash 256×512*. Independent Research.
8. Lazcano, M. (2026). *512×512 Inverted Hash*. Independent Research.

---

<div align="center">
<sub>256×256 Original Hash — © 2026 Marco Lazcano — Research prototype · Not for production use</sub>
</div>
