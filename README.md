# DASE7506 MP2 — frozen V6 selector

**Official test score: 388/467 = 83.08351177730192%.** This is an improvement over the historical V3.1 score of 385/467 = 82.4411%. It does **not** meet the requested score of at least 84.58%. The threshold requires **395/467 = 84.58244111349036%**, seven more correct choices.

The submitted method for this version is `code/submission_v6.py`; its matching checkpoint is **the complete `code/runs/final/` directory**. Do not pair this score with a historical method or checkpoint. Validation accuracy, 764/798 = 95.7393%, is a development result and must not be used as the self-reported test score.

## Reproduce without training

The course recommends Python 3.12. This release was tested on Python 3.13.5, Windows 11, Intel Core i7-12700H. Python 3.12 and the reference Xeon were not tested. From `code/`:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
$env:OMP_NUM_THREADS='4'
$env:OPENBLAS_NUM_THREADS='4'
$env:MKL_NUM_THREADS='4'
$env:NUMEXPR_NUM_THREADS='4'
python check_data.py
python predict.py --method submission_v6.py --checkpoint runs/final --split test --output runs/reproduced-v6-predictions.json
python score_predictions.py --split test --predictions runs/reproduced-v6-predictions.json --output runs/reproduced-v6-score.json
```

Expected official `accuracy`: **83.08351177730192**. The official numeric parser, data checker, prediction runner and scorer are unchanged. The method supports new bank IDs, candidate orders and ordinary bank sizes through the same interface and the runner's audit/data-directory options. Inference reads no references or saved predictions, requires no retraining, and makes no network/API call.

## Measured limits and evidence

| Item | V6 result |
|---|---:|
| Official test correctness | 388 / 467 |
| All validation correctness | 764 / 798 |
| Hard validation correctness | 321 / 355 |
| Singleton validation correctness | 86 / 114 |
| All uncompressed inference assets | 13,031,770 bytes = 12.428 MiB |
| Fresh prediction plus independent scoring | 9.694 s locally |
| Inference threads | 4 total limit |
| Report | 7 pages |

The local result is below the 16 MiB and 30-second limits. Reference-machine eligibility remains for the instructor to verify. Dependency versions are pinned; CPU inference was tested with PyTorch 2.7.1+cpu.

Evidence under `code/runs/`:

- `v6-test-score.json`: official test score.
- `v6-resources.json`: complete external timing, including startup and loading.
- `v6-validation-score.json` and `v6-checks.json`: validation and new-ID/permutation checks.
- `frozen_v6.json`: frozen asset bytes, hashes, inference source hashes and development selection. `test_evaluated=false` records the state **at freeze time**, before the separate official scoring run.
- `v6-reliability.json`: validation answer-group confidence and paired bootstrap analysis.
- `v5-cpu-training-smoke.json` and `v6-cpu-training-smoke.json`: successful selected-extension CPU retraining records.

`python evaluate_v6.py` verifies the frozen hashes and reproduces a fresh timed official evaluation. `python verify_v6.py` checks hashes and compares new-ID/permuted validation predictions; create `runs/v6-validation-predictions.json` with the official runner first if that file is absent. The released prediction JSONs are scoring evidence and are not inference assets or model inputs.

## Method and training

V6 combines six representation comparison networks, two out-of-fold ranking trees, arithmetic repair evidence, one question/operation alignment network, one singleton-trained pairwise network and one low-rank semantic ranking tree. Shared weights and symmetric comparisons avoid position and ID features. The final branch adds a training-fitted PCA projection and uses hard training groups with singleton weighting.

Only supplied training data fits model parameters, vocabularies, projections and input statistics. Validation selects epochs, architectures, score-dispersion scales and fusion weights. Earlier public test aggregates were known during later development; test references and per-example test errors were not inspected for development. The public test is exposed and is not claimed to be an untouched holdout. The report discusses approximate cross-fitting, validation reuse, transfer failures and uncalibrated confidence.

See [TRAINING.md](TRAINING.md) for exact selected-extension commands, full baseline rebuilding and research dependencies. CPU retraining for both selected extensions was tested successfully. Released neural/semantic extension weights were trained on one RTX 3060 Laptop GPU; cross-device retraining can differ. Frozen-checkpoint score reproduction uses CPU and needs no retraining.

Rejected methods and their settings are retained under `code/v4/records/`; no rejected checkpoints are needed for inference. Later seed averaging, PCA dimension expansion, supervised projections, lexical/geometry features and more singleton-focused ranking did not beat the selected validation criterion and received no additional test evaluation.

## Disclosure and submission status

OpenAI Codex substantially assisted with architecture/features, implementation, training, experiment selection, analysis and report preparation. The student must review and understand the work. The supplied course evaluation tools were reused unchanged. GSM8K-derived data retains the supplied MIT license and attribution; course candidates and representations were prepared using Qwen3-0.6B. No external datasets or additional pretrained weights were used. In-domain language pretraining experiments used only supplied training texts and were excluded from inference.

**No public repository, checkpoint URL or submission issue has been created.** Student ID and a public destination have not been provided. Publish this exact code version with its matching `code/runs/final/`, then create the issue prepared by the [MP2 website](https://xudongwu-0.github.io/courses/dase7506/mp2/) before **31 October 2026, 23:59 UTC+8**. Preparing the website link alone is not a submission. For V6 the self-reported score must be **83.08351177730192**.

The task rules are documented in the [course guide](https://github.com/xudongwu-0/DASE7506-MP2/blob/main/guide/GUIDE.md) and [official running instructions](https://github.com/xudongwu-0/DASE7506-MP2/blob/main/code/README.md). Historical score records refer to earlier frozen checkpoints, which are not all included in this release. Only `runs/frozen_v6.json` and `submission_v6.py` identify the current score/checkpoint pair.
