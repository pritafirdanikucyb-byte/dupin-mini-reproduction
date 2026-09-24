# DUPIN-Mini: A Small-Scale Reproduction

A course/independent-study reproduction of the core claim from:

> Chanwoo Bae, Hailun Ding, Shiqing Ma, Xiangyu Zhang. **"DUPIN: Attack
> Learning Is Still Needed! Demonstrating Few-Shot after Unsupervised
> Pretraining Is A Nimble Forensics Learner."** *35th USENIX Security
> Symposium (USENIX Security 26)*, 2026.

**Goal:** verify, at small scale, that unsupervised pretraining on benign
provenance graphs + few-shot fine-tuning on a handful of labeled attack
examples outperforms (a) training from scratch and (b) one-class outlier
detection — without attempting a full-scale, exact replication (the
original paper uses up to 7.3TB of logs and 25 APT campaigns).

This is explicitly a **proof-of-understanding reproduction**, not a
faithful replication. Scope, simplifications, and known limitations are
documented in the notebook and in the [Limitations](#limitations) section
below.

---

## What's in this repo

| File | Description |
|---|---|
| `dupin_mini_reproduction.ipynb` | Full pipeline: data loading, GAT pretraining, few-shot fine-tuning, baselines, evaluation. Designed to run on Kaggle (GPU + internet enabled). |
| `results/dupin_mini_results.csv` | Final metrics (TPR, TNR, AUROC) for all three approaches. |
| `results/dupin_mini_comparison.png` | Bar chart comparing the three approaches. |
| `report/` | Written summary report (motivation, problem statement, key ideas, evaluation, strengths/weaknesses). |

## Dataset

Real DARPA Transparent Computing **Engagement 3, CADETS** provenance data
(Backdoor Nginx attack — the same campaign as Attack ID 1 in DUPIN's
Table 2), obtained via **pre-processed graph files distributed by the
MAGIC repository** (one of DUPIN's own baselines) rather than the raw CDM
logs, to keep this reproduction tractable within a one-week timeline. See
[Data & Ground Truth Provenance](#data--ground-truth-provenance) below.

## Key result

| Approach | TPR | TNR | AUROC |
|---|---|---|---|
| **DUPIN-mini (pretrain + few-shot)** | 0.967 | 0.967 | **0.987** |
| No-pretrain (ablation) | 1.000 | 0.867 | 0.903 |
| One-class baseline (Isolation Forest, proxy for MAGIC/AirTag) | 1.000 | 0.867 | 0.930 |

Directionally consistent with the original paper's Table 3 (DUPIN beats
one-class baselines) and Table 7 (pretraining improves AUROC), at a much
smaller scale. Full discussion in the report.

## Limitations

- Single attack campaign (Backdoor Nginx) vs. the paper's 25 campaigns
  across 4 dataset sources.
- 2-layer/32-dim GAT vs. the paper's 7-layer/128-dim.
- No custom path tokenizer (Section 3.1 of the paper) — node "names" here
  are 6 structural type classes, not semantic file/process names.
- No cross-dataset-source validation (Section 4 of the paper) — pretraining
  corpus and test attack both come from CADETS E3.
- Subgraphs built via a single-center-node random walk (paper's Appendix E
  algorithm, γ=40/δ=100), not the paper's full node-coverage iteration.
- Ground truth comes from MAGIC's own node-level labeling methodology, not
  DUPIN's own.

## Data & Ground Truth Provenance

This reproduction avoids downloading and parsing raw DARPA CDM logs
directly (multi-GB Google Drive dump, no dates in filenames, requires a
Java toolchain) by reusing **pre-processed, ground-truth-labeled graph
files that the MAGIC paper's authors distribute in their own GitHub
repository** — since MAGIC evaluates on the same DARPA TC E3-CADETS
tracepoint as a baseline in the DUPIN paper.

Chain of provenance:
```
DARPA Transparent Computing Program (data source)
  -> MAGIC (Jia et al.) preprocesses + labels it, ships graphs.zip in their repo
    -> this reproduction loads that file directly
```

## Citations

If you build on this reproduction, please cite the original works it is
based on:

```bibtex
@inproceedings{bae2026dupin,
  title     = {DUPIN: Attack Learning Is Still Needed! Demonstrating
               Few-Shot after Unsupervised Pretraining Is A Nimble
               Forensics Learner},
  author    = {Bae, Chanwoo and Ding, Hailun and Ma, Shiqing and Zhang, Xiangyu},
  booktitle = {35th USENIX Security Symposium (USENIX Security 26)},
  year      = {2026}
}

@inproceedings{jia2024magic,
  title     = {MAGIC: Detecting Advanced Persistent Threats via Masked
               Graph Representation Learning},
  author    = {Jia, Zian and Xiong, Yun and Nan, Yuhong and Zhang, Yao and
               Zhao, Jinjing and Wen, Mi},
  booktitle = {33rd USENIX Security Symposium (USENIX Security 24)},
  pages     = {5197--5214},
  year      = {2024}
}

@inproceedings{ding2023airtag,
  title     = {AirTag: Towards Automated Attack Investigation by
               Unsupervised Learning with Log Texts},
  author    = {Ding, Hailun and Zhai, Juan and Nan, Yuhong and Ma, Shiqing},
  booktitle = {32nd USENIX Security Symposium (USENIX Security 23)},
  pages     = {373--390},
  year      = {2023}
}

@inproceedings{cheng2024kairos,
  title     = {Kairos: Practical Intrusion Detection and Investigation
               Using Whole-System Provenance},
  author    = {Cheng, Zijun and Lv, Qiujian and Liang, Jinyuan and
               Wang, Yan and Sun, Degang and Pasquier, Thomas and Han, Xueyuan},
  booktitle = {2024 IEEE Symposium on Security and Privacy (SP)},
  pages     = {3533--3551},
  year      = {2024}
}
```

**Dataset:** DARPA Transparent Computing Program, Engagement 3 (CADETS),
distributed via [darpa-i2o/Transparent-Computing](https://github.com/darpa-i2o/Transparent-Computing).

**Pre-processed data source:** [FDUDSDE/MAGIC](https://github.com/FDUDSDE/MAGIC)
(`data/cadets/graphs.zip`), used under the repository's stated license.

**Ground-truth cross-reference:** [ubc-provenance/kairos](https://github.com/ubc-provenance/kairos)
(`DARPA/CADETS_E3/evaluation.py`, `DARPA/CADETS_E5/cadets5_graph_learning.ipynb`)
was consulted to cross-check attack time windows and entity keywords against
the official DARPA ground-truth report.

## Author

Firdani Prita, Korea University Sejong Campus, Theory of Digital Forensic Course — coursework /
independent-study reproduction, 2026.

## License

This reproduction code is released under the MIT License. It depends on
and redistributes no data from DUPIN's own repository; the CADETS E3
graph data is obtained from MAGIC's repository under its own license terms.
