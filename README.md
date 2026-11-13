# TASCA
Target-Calibrated Input Context Selection for Multivariate Time-Series Foundation Model Forecasting

TASCA selects target-only, native multivariate, or source-augmented input context for a frozen forecasting model using pre-test calibration windows. This repository packages the corrected, frozen experimental implementation; it does not introduce a new selector or retune the paper's rule.

## Start in PyCharm

Select Python 3.10 (`freqrank_clean` on the authors' workstation). Open the repository directory. Every top-level Python entry point has a no-argument Run action.

1. Read `DATASETS.md` and provide the datasets and local models. Optional private overrides go in `local_config.json`, excluded from Git and the release archive.
2. Run `verify_repository.py`: 20 unit/real-data protocol tests and archived core-table regression. This does not run a full benchmark.
3. Run `smoke_models.py`: real ETTh1/ETTh2 inputs through Chronos-2 and Moirai-2, all three input contexts, with cached-repeat equality checks.
4. Run `run_experiments.py`: all corrected core studies, horizon sensitivity, Moirai-2, timestamp-aligned cases, independent-domain validation, synchronized-context evaluation, and matched LoRA. Change `STUDIES` at the top to run a subset. The core suite must finish before the full LoRA comparison.
5. Run `reproduce_tables.py`. Default `USE_REFERENCE=True` reconstructs the released core statistics; change to False for new core outputs. Each experiment engine also writes its own case, summary and comparison tables. `run_fresh_analysis.py` produces additional target-level sensitivity/admission analyses after the full core and independent suites finish.
6. Run `create_result_figures.py` for the four archived numerical result figures. These explicitly use the bundled reference tables, not fresh experiment results. Logical diagrams and the graphical abstract are not experimental outputs.

After all fresh runs and `run_fresh_analysis.py` finish, `create_fresh_result_figures.py` generates the corresponding figures from new results only, without fallback to reference data.

`smoke_lora.py` separately exercises the actual 100-step LoRA training and evaluation path for three contexts of one pair. It does not substitute for the 66-adapter comparison.

## Environment

`requirements-observed.txt` records the package versions in the tested research environment, not a solver-tested lockfile for every operating system. Python 3.10 and a CUDA-capable NVIDIA GPU are required by the frozen inference settings. Install the matching PyTorch CUDA build first; the remaining packages can be installed through PyCharm's interpreter package manager. Chronos-2 requires `chronos-forecasting` 2.x; the tested version is 2.3.0. Moirai requires `uni2ts` 2.0.0. LoRA additionally uses `peft` and `accelerate`. A fresh-environment installation has not been verified in this release.

## Scientific protocol

Core: 22 ordered source/target pairs, 10 target datasets, three chronological placements; first 7,000 rows, training-only ranking/scaling, context 96, horizon 24, up to 24 channels per domain. Leading missing values are not backward-filled; only observed forecast labels are scored. Additional horizons are 12/48/72. The raw-width threshold 32, dispersion weight 0.25 and source-admission margin 0.05 remain frozen.

Main inference uses equal-target macro averages (10 targets). Independent results use equal-collection macro averages (5 collections); synchronous validation uses 3 collections. Do not substitute pair-weighted summaries for these manuscript estimates. The original engines also export historical pair-weighted summaries for auditability. `reproduce_tables.py` makes the distinction explicit. Oracle rows are retrospective diagnostic references, not deployable selectors. Pretraining-data overlap is not certified absent.

## Resume and provenance

Predictions/window metrics use SQLite transactions; completed LoRA jobs use atomic JSON records. On interruption, rerun the same entry point: completed work is reused, while an interrupted LoRA job restarts from its fixed seed. Summaries refresh at case/replication boundaries; per-window predictions are durable. Code/model fingerprint guards prevent silently combining changed settings through the top-level launcher. Preserve old output directories and use a clean repository copy for changed experiments. Run only one writer per study.

`PROVENANCE.json` records original source and packaged hashes. `reference/` contains existing aggregate metrics, not raw series or model weights. Neither the old research directories nor their Python modules are needed by this repository. The private local configuration may reference external data files; public users provide their own data copies.

## Release status

See `VERIFICATION.md` for what was actually executed. Full experiment code is included; packaging is not a claim that every full benchmark was rerun after migration. Published reference results are clearly separated from newly executed smoke tests. The package has not been uploaded to GitHub. Before public release, confirm institutional/author permission, choose a code license, and replace bibliographic placeholders with the final accepted citation. See `LICENSE_STATUS.md`.
