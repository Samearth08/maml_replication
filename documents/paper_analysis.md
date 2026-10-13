# Implementation notes: Few-Shot Bearing Fault Diagnosis Based on Model-Agnostic Meta-Learning

**Authors:** Shen Zhang, Fei Ye, Bingnan Wang, Thomas G. Habetler.  
**Version examined:** arXiv:2007.12851v4, 24 June 2021.  
**Sources:** [abstract](https://arxiv.org/abs/2007.12851), [full text](https://arxiv.org/html/2007.12851v4), [PDF](https://arxiv.org/pdf/2007.12851).

This is an independently written implementation of the method. It does not copy the authors' or a third party's implementation. Numerical paper values below are reference measurements, not this project's measurements.

## Facts needed for implementation

| Item | Paper specification | Implementation |
| --- | --- | --- |
| Datasets | CWRU for artificial faults; Paderborn for artificial-to-real | CWRU only |
| Classes | Healthy; ball, inner-race and outer-race faults at three diameters | Same ten classes and numbering |
| Operating loads | 1, 2, 3 HP | Same three loads |
| Signal | Two sensors, 2,048 points each, nominally 12 kHz | Synchronized drive-end (DE) and fan-end (FE) windows |
| Input | 4,096 values reshaped to 64 x 64 | DE then FE, row-major reshape, one input channel |
| Segmentation | 1,980 segments/class reported; stride unspecified | Non-overlapping 2,048-point temporal windows |
| Normalization | Not specified | Per-example joint z-score, explicit assumption |
| CNN | Four Conv(3x3,64)-BN-ReLU-MaxPool(2x2) blocks | Same block sequence; padding=1; flatten then N-output linear head |
| Inner update | One gradient step; initial/fixed alpha=0.01 | Same; all CNN parameters adapted |
| Learned rates | Layer- and step-dependent rates | One scalar per parameterized layer and step; weight/bias share rate |
| Outer update | Adam selected by ablation | Adam; beta=0.001 is an assumption, not a recovered paper value |
| Meta-batch | 25 training tasks; one test task | 1 training task; one test task (CPU reduction) |
| Training duration | 1,500 iterations | 100 iterations per seed (CPU reduction) |
| Training data | Nine sampled examples per training class for Table V | Nine total: five support + four query per episode |
| Testing | N-way 1-shot and 5-shot; repeated rounds | 3/5-way, 5-shot; 30 episodes per seed, 15 queries/class |
| Primary split | Last N classes held out | 3-way: train 1–7/test 8–10; 5-way: train 1–5/test 6–10 |
| Repetitions | Ten rounds for Table V; other sections describe five/ten repeats | Three independent training seeds; sample SD across seed means |

References within the paper: class order and signals are in Section III-B/Table I; model and training settings in III-C; few-shot class splits and main results in IV-B/Table V.

## Details not resolved by the article

The paper does not fully specify segment offsets/stride, sensor concatenation order, signal scaling, padding, precise head dimensions, outer learning rate, per-task query count, batch-normalization statistics policy, exact file IDs/orientation, or the precise episode aggregation underlying every reported deviation. Its nine-training-example pool and five-shot support require a choice of disjoint training queries; this implementation uses the four remaining examples. These are declared decisions rather than claims of exact reconstruction.

The class table labels its diameter column as “mils” but gives values 0.007, 0.014 and 0.021; the official CWRU source uses **inches**, corresponding to 7, 14 and 21 mils. This project uses inches in descriptions and file mappings.

Normal baseline files 98–100 are treated as 48 kHz and anti-aliased to 12 kHz on both equal-length channels. The CWRU baseline table does not give per-channel sampling rates; this assumption is recorded in `source_manifest.csv`, not inferred from waveform lengths. Fault files come from the official 12k drive-end page. Only outer-race defects centered at 6 o'clock are selected. File 99 also contains X098 signals: the loader explicitly selects X099 to avoid duplicating file 98.

## Reference accuracy from Table V

| Experiment | Fixed-rate MAML | Learnable-rate MAML |
| --- | --- | --- |
| 3-way 5-shot | 85.53 +/- 1.07% | 90.36 +/- 1.38% |
| 5-way 5-shot | 59.86 +/- 1.04% | 79.98 +/- 1.17% |
| 3-way 1-shot (optional; not part of submitted main runs) | 82.73 +/- 3.12% | 88.39 +/- 2.95% |

The submitted primary experiments use learnable rates, so they are compared with the corresponding column. Fixed-rate support is implemented and exercised separately in verification; no fixed-rate benchmark result is claimed unless it has actually been trained and saved.

## Leakage control and interpretation

Split **classes** before meta-training; source files cannot cross the meta-training/meta-test boundary within an experiment. Test classes supply support labels only for task-local adaptation. Query labels are used only for scoring. Within each test recording, support comes from the first temporal half and queries from the second, with one whole window purged between them. Normalization uses only one example, and batch normalization uses support statistics. The model resets to the learned initialization for every test episode.

Windows within a class still come from the same physical bearing and rig, and repeated episodes reuse the finite pool. This is not an independent-bearing or external-domain test. Splits for 3-way and 5-way are separate experiments; some class roles necessarily differ across them. A reduced computation budget and different segmentation/statistics make differences from the paper descriptive, not controlled estimates of algorithmic superiority or failure.
