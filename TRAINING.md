# Training and research reproduction

Run commands from `code/` in an environment containing `requirements.txt`. Limit OMP/OpenBLAS/MKL/NumExpr and Torch to four threads. Training writes a fresh output; do not overwrite `runs/final/`. The frozen checkpoint is the claimed-score reproduction target. Different devices or library versions can change retrained weights and scores.

## Selected extensions, CPU supported

```powershell
python train_v5.py --device cpu --output runs/retrained_v5
python train_v6.py --device CPU --output runs/retrained_v6
```

These two commands were executed successfully locally. They independently rebuild one extension while retaining the supplied frozen earlier components. `train_v6.py` regenerates its training-only out-of-fold prior cache when missing. The retained `runs/v2/oof_stack_train.npy` is a training cache, never read by inference; `train_stack.py` regenerates it after original baseline training.

To rebuild both extensions together, use the first output as the second command's baseline:

```powershell
python train_v6.py --device CPU --baseline runs/retrained_v5 --output runs/retrained_both
```

The released V5 component used seed 3906, seven epochs, 684 singleton training groups, 64 hidden dimensions, AdamW 0.0004, decay 0.1 and batch size 96. Its normalisation fits hard valid training candidates, not validation. The released V6 ranker used seed 4103, 64 PCA components, 800 training trees pruned to the selected first 125, depth 5, PairLogit, learning rate 0.025, L2 30, 32 borders and singleton group weight 3. Both fixed fusion weights are 0.25; score dispersions are recalculated on validation inputs without reading validation references during these fixed-setting retraining commands.

GPU options are `--device cuda` for `train_v5.py` and `--device GPU` for `train_v6.py`. Original extension training used one NVIDIA RTX 3060 Laptop GPU (6 GB) with PyTorch 2.7.1+cu126 for neural training. CPU inference uses the ordinary CPU dependency environment. No pretrained model download is used.

## Rebuild earlier components

`train_all.ps1` runs the baseline, family comparisons, out-of-fold fitting and validation selection. It fits train inputs/references and uses validation for selection, then freezes the original prior. Its 24-step list is an executable ordering of historical development. It never runs official test scoring automatically. Preserve a separate checkout/output if you want to replay the whole baseline workflow.

`train_v4.ps1` then runs `v4/build_alignment_inputs.py`, `train_alignment.py`, `select_alignment.py` and `freeze_alignment.py`, writing `runs/retrained_v4/`. This rebuilds the alignment component against the supplied baseline weights. Use that directory as `train_v5.py --baseline runs/retrained_v4` when rebuilding the whole chain.

Training reference operation profiles supervise an auxiliary target; candidate profiles at inference use candidate text only. All parsers interpret restricted arithmetic syntax and do not execute text.

## Research studies and dependencies

For replaying unselected experiments, work in `code/v4/`. First run `prepare_research.py`, which rebuilds train/validation candidate profiles, training reference profiles and the V3.1/V4/V5 validation score caches from the released frozen components. It does not open test inputs or references. Research scripts may overwrite their own generated caches/checkpoints, so replay in a fresh copy.

| Study | Run in this order after preparation |
|---|---|
| Common program prediction | `derive_programs.py`, `train_program.py` |
| Compositional program matching | `derive_programs.py`, `derive_compositional.py`, `train_compositional.py` |
| Reference profile ranking | `train_profiles.py`, `train_profile_rank.py` |
| Correct-candidate profile ranking | `train_candidate_expected.py`, `train_candidate_rank.py` |
| Alignment search | `train_alignment.py`, `select_alignment.py` |
| Sequence verifier | `sequence_data.py`, `train_sequence.py` |
| Memory sequence verifier | `sequence_data.py`, `train_sequence_memory.py` |
| Dynamic GPU verifier | `sequence_data.py`, `build_v3_oof.py`, `train_dynamic_gpu.py` |
| In-domain MLM and fine-tuning | `sequence_data.py`, `build_v3_oof.py`, `build_mlm_data.py`, `train_mlm.py`, `train_mlm_finetune.py` |
| Small-network training-only CV | `train_quality_cv.py`, `evaluate_quality.py` |
| Pairwise comparison searches | `train_relational.py`, `train_direct_pair.py`, `select_complementary.py` |
| Rich-feature pairwise search | `train_rich_pair.py` |
| Semantic ranker | `build_v3_oof.py`, `train_semantic_residual.py` |
| PCA ensemble comparison | `train_semantic_residual.py`, `train_semantic_ensemble.py`, `select_semantic_ensemble.py` |
| Supervised projection comparison | `build_v3_oof.py`, `train_supervised_projection.py` |
| Geometry/coverage and focus comparisons | `train_semantic_residual.py`, `train_geometry.py`, `train_focused_semantic.py` |
| Rule evidence | `evaluate_semantics.py`, `train_experts.py` |
| Validation reliability | `train_direct_pair.py`, `select_complementary.py`, `train_semantic_residual.py`, `analyze_v6_reliability.py` |

GPU-named, MLM, direct-pair and recent tree research scripts use the GPU path; the fixed selected-extension scripts above provide tested CPU alternatives. Completed research records are in `v4/records/`, including rejected results and elapsed family times. The stopped CPU cross-encoder feasibility trial is labelled incomplete. Search records and baseline/current source code are included; large rejected checkpoints are omitted and regenerate from these scripts.

The next-stage out-of-fold tree prior reuses neural out-of-fold features and is not a fully nested cross-fit of the entire V5 pipeline. Its training prior is based on V3.1 while validation/inference feature priors use V5. This approximation is disclosed rather than being treated as independent holdout evidence.

## PDF source

`report_source.py` regenerates the seven-page report with ReportLab. Run it from this release root after installing `requirements-report.txt`; its output is `REPORT.pdf`. It is separate from inference and training dependencies.
