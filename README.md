# Few-Shot Bearing Fault Diagnosis Using MAML

**Samarth Sharma** · samarth.011@nmims.in

A practical reproduction of [Zhang et al., Few-Shot Bearing Fault Diagnosis Based on Model-Agnostic Meta-Learning](https://arxiv.org/abs/2007.12851), using real CWRU data and PyTorch.

## Submission files

- **[Jupyter notebook](src/MAML_Bearing_Reproduction.ipynb):** the complete implementation, explanations, experiments, verification and saved outputs in one file.
- **[Replicated research paper](documents/Replicated%20paper.pdf):** IEEE journal-style PDF with actual measured results.
- **[Presentation](PPT/ML%20ppt.pptx)** and **[original research paper](documents/Base%20paper.pdf)**.
- [Replication report](documents/supplementary/Replication%20report.pdf), [paper analysis](documents/paper_analysis.md), and [replication checklist](results/replication_checklist.md).
- [Results comparison](results/tables/paper_vs_ours.md) and [per-seed results](results/tables/per_seed_results.csv).

The notebook uses code cells with brief comments, one import block, and step-by-step task sampling and adaptation code. It has no Markdown heading or explanation cells. The saved outputs are retained from the completed runs; the readability edits were checked separately so that graphs, metrics and timings remain unchanged.

The notebook contains the code directly. It does not import or create any project Python scripts. It needs the installed packages and real dataset to run. The supplied results and checkpoints allow it to validate and reuse the completed experiments.

## Open and run

Use Python 3.10 or newer. Clone or download this repository, then run from its project folder:

```powershell
python -m pip install -r requirements.txt
python -m jupyter notebook src/MAML_Bearing_Reproduction.ipynb
```

Set `DATASET_PATH` in the first code cell to your CWRU folder and select **Restart Kernel and Run All Cells**. The default dataset location is `../CWRU_raw`. Follow [dataset instructions](data/README.md) for the exact 30 recordings. Raw recordings are excluded from the submission.

The notebook first runs real-data debug checks, then runs the two primary experiments. `RERUN_TRAINING=False` reuses completed runs only after checking configurations and data hashes. Missing results are trained automatically. Set `RERUN_TRAINING=True` to retrain all six runs. Saved measurements total about 43 minutes for the original six training/evaluation runs on this machine; runtime varies. Debugging and verification also take time.

All settings are in the first cell's `Config` class: N-way, K-shot, query count, inner steps, learning rates, training iterations, meta-batch size and evaluation tasks. Edit `SEEDS` to change seeds. The primary experiment cells explicitly use 3-way/5-way and 5-shot; update those cells for other experiments. Use a new results folder for experiments whose settings differ from the supplied runs. Update the paper after generating new results.

To execute without the browser interface:

```powershell
python -m jupyter nbconvert --to notebook --execute --inplace --ExecutePreprocessor.timeout=3600 src/MAML_Bearing_Reproduction.ipynb
```

Jupyter needs local kernel sockets; use a normal terminal if a restricted sandbox blocks them.

## Notebook contents

1. Introduction and paper methodology
2. Configuration and dependency imports
3. Official dataset file mapping and optional downloader
4. Signal preprocessing and sample visualizations
5. Four-block CNN
6. N-way task sampling and full second-order MAML
7. Evaluation helpers, meta-training and real-data debug checks
8. 3-way 5-shot: three seeds
9. 5-way 5-shot: three seeds
10. Metrics, confusion matrices, curves and paper comparison
11. Discussion and limitations
12. Final integrity checks and conclusion

## Method

```text
DE 2048 + FE 2048 samples
  -> per-example z-score -> 64 x 64 input
  -> four Conv(3x3, 64)-BN-ReLU-MaxPool blocks
  -> N-way linear classifier
  -> support adaptation -> query loss -> meta-gradient -> Adam update
```

The implementation uses PyTorch autograd directly, with learnable layer/step inner rates. No black-box MAML package is used. Train/test classes and files are disjoint within each experiment. Test support and query windows use separate temporal pools and a purged window; query batch-normalization uses support statistics; signal z-score normalization is per example.

## Actual results

| Experiment | Seed 1 | Seed 2 | Seed 3 | Mean ± sample SD |
| --- | --- | --- | --- | --- |
| 3-way 5-shot | 95.33% | 93.85% | 92.96% | 94.05 ± 1.20% |
| 5-way 5-shot | 58.67% | 52.44% | 62.98% | 58.03 ± 5.30% |

The paper's learnable-rate references are 90.36 ± 1.38% and 79.98 ± 1.17%, respectively. Differences are +3.69 and -21.95 percentage points. This is a contextual comparison: our budget is 100 iterations with one task per meta-batch, versus the paper's 1,500 iterations and 25 tasks. Our segmentation, normalization and evaluation assumptions also differ. Repeated query windows are not independent bearing trials. See the [replication checklist](results/replication_checklist.md).

## Project structure

```text
maml_replication/
├── data/                 Dataset instructions and source manifest
├── documents/            Original paper, replicated paper and supporting reports
├── PPT/                  Project presentation
├── results/              Figures, tables, metrics, checkpoints and checklist
├── src/
│   └── MAML_Bearing_Reproduction.ipynb
├── README.md
└── requirements.txt
```

There is exactly one code file: the notebook. The [file inventory](documents/supplementary/FILE_MANIFEST.txt) lists all submission artifacts. Submit the ZIP, or the notebook and paper as separate uploads when requested. The notebook also runs independently of the folder layout: launch it in a writable directory, configure `DATASET_PATH`, and it creates its results there. Without supplied cached runs, it trains fresh experiments.

## Limitations and references

Only the two 5-shot experiments were benchmarked. Fixed-rate MAML is checked for gradients, but no fixed-rate benchmark is claimed. The optional 1-shot and Paderborn transfer experiments were not run. Model padding/head details, signal scaling, normal-signal sampling rate, outer learning rate and BN behavior include documented assumptions. No test-based checkpoint selection was used.

- [Original bearing diagnosis paper](https://arxiv.org/abs/2007.12851)
- [CWRU Bearing Data Center](https://engineering.case.edu/bearingdatacenter/download-data-file)
- [Finn et al., original MAML](https://proceedings.mlr.press/v70/finn17a.html)
