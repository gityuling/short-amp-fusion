# Compact Multiview Fusion for Short Antimicrobial Peptide Identification

Version 1.0.0 · [Zenodo archive](https://doi.org/10.5281/zenodo.24358016) · [GitHub](https://github.com/gityuling/short-amp-fusion)

Download and extract **short-amp-fusion-1.0.0.zip**. Run the commands below from the extracted directory. This release contains the study code, recorded parameters, source download registry, sequence-free test scores and case evidence. Third-party datasets and encoder weights are retrieved from their original sources.

## Verify the main results on a CPU

With Python 3.10 or 3.11 and NumPy 1.26.4 installed:

```sh
python verify.py
```

The script recomputes the mean of the three seed-specific F1 scores for the five models and the matched ProtBERT-plus-descriptors ablation, followed by the paired 10,000-replicate cluster interval. It also checks the 12 recorded agent reports against their six evidence packets per case. It requires no network, model training, or API credentials.

| Comparison on 1,400 official test peptides | Mean F1 difference |
| --- | ---: |
| Full fusion (80.79%) versus locally retrained PPsAMP (77.03%) | +3.76 percentage points (95% paired cluster interval +1.45 to +6.12) |
| Full fusion versus ProtBERT plus explicit descriptors (79.92%) | +0.88 percentage points |

Per-seed metrics are in reference/S16. S17 reports both trapezoidal PR area and average precision. S01–S03 cover the ablations and uncertainty. S10–S11 distinguish numerical checks from the human assessment of the agent reports. S15 records the experiment settings.

## Retrain on the official PPsAMP split

Use Linux, Python 3.10 and a CUDA-capable GPU. The original Python package versions are in reproduction/requirements.lock.txt. Install the matching PyTorch wheel using the CUDA 12.8 PyTorch index. Run in a fresh extraction:

```sh
python -m pip install torch==2.7.1 --index-url https://download.pytorch.org/whl/cu128
python -m pip install -r reproduction/requirements.lock.txt
GIT_LFS_SKIP_SMUDGE=1 git clone https://github.com/shengxiliu/PPsAMP.git PPsAMP
GIT_LFS_SKIP_SMUDGE=1 git -C PPsAMP checkout 8efc6d78f8a2afda1766c8fcb05df4d8cfa86544
python tools/download_inputs.py --group ppsamp-data --group ppsamp-assets --root .
mkdir -p manuscript_experiments/data
python reproduction/scripts/extract_features.py
for seed in 42 43 44; do python reproduction/scripts/run_baseline.py --mode train --seed "$seed" || exit 1; done
python optimization/prepare_esm_cache.py
python optimization/prepare_test_cache.py
python optimization/run_phase5.py
python optimization/run_phase8.py
python manuscript_experiments/scripts/run_core.py
```

The downloader checks byte counts and SHA-256 hashes. The official train/validation/test splits contain 4,724/1,398/1,400 peptides. The ESM-2 revision is pinned in optimization/phase8_protocol.json. The two phase scripts generate the fitted transforms and heads reused by run_core.py. Core predictions are written under manuscript_experiments/results/core/.

To retrain the other three comparators on these same splits:

```sh
mkdir -p manuscript_experiments/sources
git clone https://github.com/BigDataBiology/macrel.git manuscript_experiments/sources/macrel
git -C manuscript_experiments/sources/macrel checkout 8c1f7321618ccde2a28b960d32525620d2c737e3
curl -fL https://www.ebi.ac.uk/europepmc/webservices/rest/PMC7256447/fullTextXML -o manuscript_experiments/sources/deep_ampep30.xml
python manuscript_experiments/scripts/run_comparators.py
```

Macrel uses the upstream feature implementation. Deep-AmPEP30 and RF-AmPEP30 are paper-based reimplementations. See manuscript_experiments/protocols/comparators.json for the preserved implementation choices.

## Supporting analyses

- **External benchmark.** The download groups battle-retrain, battle-official, and reference-predictions point to separate inputs. Use the same downloader with the desired groups. The original split uses Debian MMseqs2 15-6f452+ds-2 and libgzstream0, extracted beneath manuscript_experiments/tools/mmseqs2/unpacked. Run manuscript_experiments/scripts/battle_amp_study.py first and battle_official_eval.py for the official evaluation. The published BATTLE reference predictions come from the benchmark authors. The recorded code and tables distinguish retrained models from frozen evaluations.
- **Attribution and case analysis.** manuscript_experiments/interpretability_v1 contains the attribution and permutation analyses. For the three experimental panels, use the case-sources download group and the source map in external_agent_three_v1/DOWNLOAD_AND_REPRODUCE.txt. Extract the SEP supplementary ZIP into its sources/sep_supplements directory and convert its tables PDF with pdftotext -layout. Run prepare_cohorts.py, run_frozen_models.py and analyze_predictions.py there, then external_explanations_v2/analyze.py for case attributions and rescoring (SHAP 0.46.0; openpyxl for the source tables).
- **Agent evidence.** reference/case_checks.json.gz contains the final report and six evidence packets for each archived case. verify.py runs the original typed fact checks offline. reference/agent_protocol.json records the API settings. The original agent prompts, tool dispatch and scoring code are in external_explanations_v2/agent.py. A new live run requires regenerated case inputs, the preceding provenance step in external_agent_three_v1/agent_investigation.py and a user-provided DEEPSEEK_API_KEY. A live service can return different wording.
- **Structural context.** structure_context_pilot_v1 contains the ESMFold revision, folding settings, structure-generation code and case-level output summaries. Structures illustrate the tested positions. Experimental activity labels come from the cited source studies.
- **Access and licensing.** MIT applies to the authors' software. Upstream source code, pretrained weights and third-party data retain their original terms. The pinned registry supplies URLs, target paths, sizes and hashes. Add --list to inspect a download group or --verify-only to check an existing local copy.

The release check runs the CPU score and evidence replay. Full GPU retraining and live API generation are separate workflows.
