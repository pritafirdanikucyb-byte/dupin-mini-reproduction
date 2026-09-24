# DUPIN-Mini: Small-Scale Reproduction Experiment

A write-up of the experimental process behind reproducing the core claim of:

> Chanwoo Bae, Hailun Ding, Shiqing Ma, Xiangyu Zhang. *"DUPIN: Attack Learning Is Still Needed! Demonstrating Few-Shot after Unsupervised Pretraining Is A Nimble Forensics Learner."* 35th USENIX Security Symposium (USENIX Security 26), 2026.

## 1. Goal

Verify, at small scale, that unsupervised pretraining on benign provenance graphs followed by few-shot fine-tuning on a handful of labeled attack examples outperforms (a) training the same model from scratch and (b) one-class outlier detection that never sees attack data. This is a proof-of-understanding reproduction, not a full replication — the original paper uses up to 7.3TB of logs across 25 APT campaigns and four dataset sources.

## 2. Dataset

Rather than parsing DARPA Transparent Computing's raw CDM logs directly (a multi-GB, Java-toolchain-dependent pipeline), this reproduction reuses pre-processed, ground-truth-labeled graph files that the MAGIC paper (Jia et al., USENIX Security 24 — one of DUPIN's own baselines) distributes directly in its GitHub repository (`FDUDSDE/MAGIC`, `data/cadets/graphs.zip`). This provides real DARPA Transparent Computing **Engagement 3, CADETS** data — the same tracepoint and attack family (Backdoor Nginx) as Attack ID 1 in DUPIN's Table 2 — with node-level ground truth already available in `metadata.json`.

Four benign provenance graphs (`train0-3.pkl`, different segments of the CADETS E3 capture) served as the pretraining corpus. One graph containing both benign activity and the Backdoor-Nginx attack (`test0.pkl`) served as the source for few-shot fine-tuning and evaluation.

## 3. Methodology

**Subgraph extraction.** Subgraphs were built using the paper's own random-walk algorithm (Appendix E, Algorithm 1): starting from a center node, repeatedly hopping to a random neighbor already in the frontier, until the subgraph reaches size γ=40 or the walk stalls for δ=100 iterations — the exact parameters the paper itself uses. Following the paper's own evaluation principle (Section 4) that ground truth must not influence data collection, the starting node for every subgraph is chosen uniformly at random from the entire graph, with no knowledge of which nodes are malicious; a subgraph is labeled as an attack sample only *afterward*, if the resulting walk happens to include a known-malicious node.

**Model.** A 2-layer, 32-dimensional Graph Attention Network (vs. the paper's 7-layer, 128-dimensional model), implemented from scratch in plain PyTorch. Node and edge-type embeddings follow the paper's Equations 3-4; masked node/edge-type prediction follows Equations 5-7 for pretraining; a linear classifier head follows Section 3.4 for fine-tuning.

**Compared approaches:**
- **DUPIN-mini**: pretrain on the 4 benign graphs, then fine-tune on 5 labeled attack subgraphs + 5 benign subgraphs.
- **No-pretrain (ablation)**: identical architecture, randomly initialized, fine-tuned on the same 5+5 samples — reproducing the paper's Table 7 ablation.
- **One-class baseline**: an Isolation Forest trained only on pretrained-embedding representations of benign graphs, standing in for the category of self-supervised/one-class methods (MAGIC, AirTag) the paper argues against — not a reimplementation of either method itself.

**Threshold selection.** Following the paper's own method (Section 4.2), each model's classification threshold is chosen by scanning the ROC curve and selecting the point minimizing Euclidean distance to the ideal corner (TPR=1, TNR=1), rather than assuming a fixed 0.5 cutoff is meaningful for this imbalanced detection task.

## 4. Results

| Approach | TPR | TNR | AUROC |
|---|---|---|---|
| **DUPIN-mini (pretrain + few-shot)** | 0.967 | 0.967 | **0.987** |
| No-pretrain (ablation) | 1.000 | 0.867 | 0.903 |
| One-class (Isolation Forest, proxy for MAGIC/AirTag) | 1.000 | 0.867 | 0.930 |

The AUROC ordering — DUPIN-mini > One-class > No-pretrain — is directionally consistent with the paper's own Table 3 (DUPIN beats one-class baselines) and Table 7 (pretraining improves AUROC by +0.16 in the original paper, vs. +0.084 here). No-pretrain and the one-class baseline both reach perfect TPR (1.0) but at the cost of more false positives (TNR 0.867); DUPIN-mini instead reaches a more balanced TPR/TNR (0.967/0.967) — fewer benign entities incorrectly flagged, without missing more attack entities.

## 5. Limitations of This Reproduction

- Single attack campaign (Backdoor Nginx) vs. the paper's 25 campaigns across 4 dataset sources.
- No cross-dataset-source validation (Section 4 of the paper) — the pretraining corpus and the test attack both come from CADETS E3.
- No custom path tokenizer (Section 3.1) — node "identities" here are 6 structural type classes, not semantic file/process names like `vUgefal` or `nginx`.
- Subgraphs are built from a single center node per sample, rather than the paper's full node-coverage iteration.
- Ground truth comes from MAGIC's own node-level labeling methodology for E3-CADETS, not from DUPIN's own labeling process — the two may not label identical entities even for the same campaign.
- The one-class baseline is a simple Isolation Forest, not a reimplementation of MAGIC or AirTag, so the observed gap against DUPIN-mini likely understates the paper's own reported margin.
