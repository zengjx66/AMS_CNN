# Adaptive Multi-Size CNN: Fast Mode Decision for VVC-SCC

> **Published in DCC 2026**, p. 483 · DOI: [10.1109/DCC66757.2026.00110](https://doi.org/10.1109/DCC66757.2026.00110)

Official implementation of *Adaptive Multi-Size CNN: Fast Mode Decision for VVC-SCC*.

**Authors:** Jiaxin Zeng, Linyun Liu, Licong Ma, Wanglin Lai, Li Yu (Huazhong University of Science and Technology)

## Overview

AMS-CNN speeds up VVC **intra mode decision** by pruning the RDO candidate list:

1. A multi-size CNN predicts intra-mode probabilities for 17 CU sizes.
2. Probabilities are adjusted using the optimal modes of the top / left / top-left neighboring CUs.
3. Only modes above a threshold enter the RDO list.

On **VTM-23.11**, all-intra, QP 22 / 27 / 32 / 37, CTC SCC test set: **1.20% BDBR** for **31.35% encoding time saving** on average.

## Setup

- VTM-23.11 (patched for AMS-CNN inference)
- Python 3.8+, PyTorch ≥ 1.12, NumPy, OpenCV

```bash
pip install torch torchvision numpy opencv-python
```

## Reproduce

```bash
# 1. Build the patched VTM-23.11 encoder
cmake -S <vtm-root> -B <vtm-root>/build && cmake --build <vtm-root>/build -j

# 2. Encode each CTC sequence at QP 22 / 27 / 32 / 37
<encoder> -c <ai.cfg> -c <scc.cfg> -i <seq>.yuv -q <qp> -f <frames> > <log>.txt

# 3. Compute BDBR and ΔT against the VTM-23.11 anchor
python <eval.py> --logs <log-dir> --out results.csv
```

Replace `<...>` with the actual scripts and flags in this repo. Keep the YUV, cfg files, QPs and frame count identical for the anchor and all baselines.

## Results

Average over 14 CTC SCC sequences (BDBR: lower is better; ΔT < 0 = time saved):

| Method | BDBR (%) | ΔT (%) |
| --- | --- | --- |
| FastSCCNet (VCIP 2020) | 3.19 | -28.04 |
| WA-CNN (TMM 2024) | 2.47 | -39.15 |
| HFF-CNN (SPL 2025) | 0.38 | -20.24 |
| **AMS-CNN (ours)** | **1.20** | **-31.35** |

Per-sequence results: Table 1 of the paper.

## Citation

```bibtex
@inproceedings{zeng2026amscnn,
  title     = {Adaptive Multi-Size CNN: Fast Mode Decision for VVC-SCC},
  author    = {Zeng, Jiaxin and Liu, Linyun and Ma, Licong and Lai, Wanglin and Yu, Li},
  booktitle = {2026 Data Compression Conference (DCC)},
  pages     = {483},
  year      = {2026},
  doi       = {10.1109/DCC66757.2026.00110}
}
```

## Contact

Li Yu · hustlyu@hust.edu.cn
