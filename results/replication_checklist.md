# MAML replication checklist

**Project:** Few-Shot Bearing Fault Diagnosis Based on Model-Agnostic Meta-Learning  
**Author:** Samarth Sharma — samarth.011@nmims.in

This checklist distinguishes implemented paper methodology from practical choices and experiments that were not run. It supplements the [paper analysis](../documents/paper_analysis.md) and [execution checklist](../documents/supplementary/FINAL_CHECKLIST.md).

## Methodology and experimental coverage

| Component | Status | Evidence or difference |
| --- | --- | --- |
| CWRU fault diagnosis | Reproduced in scope | Thirty official recordings, ten health/fault classes, loads of 1, 2 and 3 HP. See [dataset guide](../data/README.md). |
| Input representation | Core method reproduced | Two synchronized 2,048-point sensor windows form 4,096 values, reshaped to 64 × 64. |
| Segmentation and normalization | Declared assumptions | Non-overlapping windows and per-example z-score; exact stride and scaling are unspecified in the paper. |
| Healthy-recording sample rate | Declared assumption | Both normal channels are treated as 48 kHz and resampled to 12 kHz. The source does not resolve per-channel rates. |
| CNN | Core architecture reproduced | Four convolution–batch normalization–ReLU–pool blocks, 64 filters each. Padding and final head dimensions are declared choices. |
| MAML | Implemented and verified | Full second-order gradients through support adaptation; query loss updates the shared initialization. |
| Learnable inner rates | Implemented | Layer/step rates initialized at 0.01; the submitted runs use one adaptation step. Weight and bias share a layer rate. |
| Outer optimizer | Partial specification | Adam; outer learning rate 0.001 is an implementation assumption. |
| Meta-training class pool | Reproduced in scope | Nine examples per class, divided into five support and four query examples per task. |
| Unseen-class split | Reproduced in scope | 3-way: classes 1–7 train, 8–10 test. 5-way: classes 1–5 train, 6–10 test. These are separate experiments. |
| Leakage prevention | Verified implementation choice | Disjoint train/test classes and files; test support/query windows separated temporally with a purged window; queries use support-derived BN statistics. |
| Training budget | Reduced | 100 iterations × one task per iteration, versus the paper's 1,500 × 25. |
| Evaluation protocol | Different | Thirty episodes per seed, 15 queries per class and three training seeds; not the paper's ten-round aggregation. |
| Primary experiments | Completed | 3-way 5-shot and 5-way 5-shot, each with seeds 1, 2 and 3. |
| Fixed-rate MAML | Implementation checked only | Finite meta-gradients verified; no full fixed-rate benchmark is reported. |
| 3-way 1-shot | Not run | Supported by the training interface; no result is claimed. |
| Paderborn transfer and other baselines | Not reproduced | Outside this practical reproduction's scope. |
| Notebook | Executed | All 21 code cells have execution counts and no error outputs. Cached completed experiments are validated before reuse. |
| Metrics and checkpoints | Verified | Metrics recomputed from predictions; saved configurations and dataset hashes checked. Seed-1 checkpoints reproduce their first two test episodes exactly. |

## Recorded results

| Experiment | Our accuracy, mean ± sample SD | Seeds |
| --- | --- | --- |
| 3-way 5-shot | 94.05 ± 1.20% | 1, 2, 3 |
| 5-way 5-shot | 58.03 ± 5.30% | 1, 2, 3 |

The [paper comparison](tables/paper_vs_ours.md) gives the reference values and differences. These comparisons are descriptive because segmentation, training budget and evaluation differ. Repeated episodes reuse correlated windows from a finite set of recordings; the prediction count is not a count of independent bearings.

## Submission evidence

- [Executed Jupyter notebook](../src/MAML_Bearing_Reproduction.ipynb)
- [Replicated research paper, IEEE journal-style PDF](../documents/IEEE_Reproduction_Paper.pdf)
- [Editable manuscript](../documents/IEEE_Reproduction_Paper.md)
- [Per-seed results](tables/per_seed_results.csv) and [paper-vs-ours table](tables/paper_vs_ours.md)
- [Machine-readable verification](metrics/verification.json)
- [Complete file inventory](../documents/supplementary/FILE_MANIFEST.txt)

Raw MATLAB recordings are excluded from the repository and submission ZIP. All research code is included in the notebook; there are no local Python-module dependencies. Dataset access is required to rerun the real-data cells.
