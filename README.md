# Can Prompting Reliably Close the Sentence-Length Dispersion Gap? Code and evaluation artifacts

Artifact repository for the paper "Can Prompting Reliably Close the
Sentence-Length Dispersion Gap?" (LP4FM @ NeurIPS 2026). Every script and
evaluation artifact the paper relies on, in source-repository layout, so the
scripts run from this directory as the working root.

## Dataset

The human articles, the generations of every leg and of the paraphrase probe, and the
18 October run records are released as one public dataset:
https://huggingface.co/datasets/burstiness-engine/sentence-length-dispersion. Its card
describes every file and gives the commands that re-score a leg into the evaluation JSON
committed here; its MANIFEST.json gives each file's row count, source dataset, source
revision and SHA-256. hf-datasets.zip in this repository is a copy of it. The dataset
references in each experiments/results/benchmark/<family>/meta.json name the private
source datasets these files were copied from.

## Paper source

- paper/lp4fm/camera-ready.pdf: the camera-ready PDF, built from main.tex at this commit.
- paper/lp4fm/main.tex: the unified LaTeX source of the camera-ready (single assembled
  file), with paper/lp4fm/references.bib, paper/lp4fm/neurips_2026.sty, and
  paper/lp4fm/build.sh (tectonic) alongside. Figure 1 is a PDF plot under
  paper/lp4fm/figures/; paper/lp4fm/make_lp4fm_figures.py redraws it from the result JSONs.

## Code (mirrors repository paths)

- bench/prompts.py: the five prompting modes, their composition rule, and a
  self-check that enforces the three-call self-refine structure and metric-free
  prompt text (Appendix A). bench/m2_exemplars.json holds the three few-shot
  exemplar articles.
- bench/run.py, bench/run_batch.py, bench/providers.py, bench/config.py,
  bench/storage.py: the API generation harness. bench/gen_vllm.py is the local
  open-weights lane (Qwen, Gemma). bench/scrub_thinking.py strips reasoning traces
  before scoring.
- experiments/burstiness_pp.py: the metric (B_n under the finite-n correction, the
  Gini robustness check, Miller-Madow segmentation entropy, memory coefficient M),
  the Goodhart guard, the multiscale profile, and a self-check. Standard library
  only; reproduced in Appendix C. experiments/burstiness_metrics.py carries the
  sentence segmentation and punctuation helpers it imports.
- experiments/ablation/clean_wikinews.py: deterministic corpus parse from Wikinews
  wikitext (Section 4.1). build_ablation_corpus.py assembles the paired corpus.
  select_m2_exemplars.py applies the median-rule exemplar selection.
- experiments/ablation/normalize_generation.py: the uniform normalization applied to
  every leg before scoring (Section 4.2), with test_normalize_generation.py.
- experiments/ablation/seed_avg_paired.py (with paired_test.py): the seed-averaged
  Wilcoxon protocol behind every reported delta (Section 4.3).
- experiments/ablation/equivalence_test.py: CI-bounded equivalence margins from the
  committed per-family CIs (Section 4.3).
- experiments/ablation/multiplicity_correction.py: the BH and Holm census
  (Section 4.3).
- experiments/ablation/contamination_probe.py: the n-gram overlap probe
  (Limitations), with test_contamination_probe.py.
- experiments/ablation/generation_hygiene.py: the markdown and title-echo audit that
  motivated normalization (Section 4.2).
- experiments/ablation/prompt_paraphrase.py: the instruction-wording robustness
  probe (Limitations).
- experiments/ablation/review_analyses.py: the supplementary analyses (human B_n
  spread, mode contrasts, raw-variance baseline, guard rates, ten-sentence floor,
  lengths) behind the appendix tables; writes experiments/results/review_analyses.json.
- experiments/test_segmenter.py: regression tests for the sentence segmenter.
- paper/gen_bench_table.py: regenerates Table 1, the per-family grids, the appendix
  tables and every number macro (paper/lp4fm/generated/) from the released evals; no
  hand-transcribed values.

## Data

- experiments/results/benchmark/<family>/seed_avg_paired.json: the compiled scored
  evaluation per family (per metric and mode: n_pairs, median paired delta,
  bootstrap 95% CI, Wilcoxon z and p, matched rank-biserial, paired dz).
- experiments/results/benchmark/<family>/meta.json: generation metadata (decoding,
  corpus, design, exclusions, provenance).
- experiments/results/benchmark/<family>/multiplicity.json: the BH and Holm census.
- experiments/results/benchmark/<family>/equivalence.json: compiled instruction-mode
  equivalence margins; regenerable for any family from seed_avg_paired.json via
  equivalence_test.py.
- experiments/results/benchmark/qwen3.6-27b/seed_avg_paired.raw-archive.json: the
  audited leg scored under the raw protocol (Section 4.2).
- experiments/results/paraphrase_robustness/: the DeepSeek paraphrase probe evals
  (18 October 2026 run, against the snapshot of the dagger-marked leg; the July 2026 evals the
  reviewed version reported are kept in july-2026/).
- experiments/results/review_analyses.json: the supplementary analyses output.
- experiments/results/contamination_probe.json: the n-gram probe output.
- hf-datasets.zip: a copy of the dataset burstiness-engine/sentence-length-dispersion
  (see Dataset above): the frozen human baseline (prompts_200_human.jsonl, 200 paired
  Wikinews articles), the 100-pair ablation corpus, the raw generations of all nine
  benchmark legs, and the 18 October 2026 DeepSeek re-run and paraphrase probe with their
  run records; reasoning text is not included. See the README inside the zip.

The human-side corpus also regenerates deterministically from Wikinews wikitext
via clean_wikinews.py; normalized generation variants regenerate from the raw
files via normalize_generation.py.

## Quick verification (run from this directory)

    python -m experiments.burstiness_pp --demo        # metric self-check
    python bench/prompts.py                           # prompt composition self-check
    python experiments/ablation/equivalence_test.py --model kimi-k2.6
    python paper/gen_bench_table.py                   # re-emit Table 1 / Figure 1

Scripts with package imports (burstiness_pp, seed_avg_paired) run module-style with
python -m from this root. Requirements: Python 3.10+. The metric and all scoring are
standard library. Generation needs bench/requirements.txt; the corpus rebuild needs
mwparserfromhell.

## Contributions

**Vittoria Lanzo** (first author) proposed the project and its research questions, and
narrowed its scope from an in-generation controller to a measurement study of prompting.
She defined the metric (the Goh–Barabási coefficient under the Kim–Jo finite-n correction,
with its Gini check, segmentation-entropy and memory companions, and the Goodhart guard),
validated it in the era-controlled ablation, and ran the analysis showing that
sentence-length variance tracks mean sentence length while B_n does not. She designed the
benchmark protocol: the paired Wikinews corpus, the five prompting modes with frozen
few-shot exemplars, the generation harness and the seed-averaged paired test. She built the
vLLM pipeline for open-weights generation and ran the Qwen3.6-27B leg on it. She wrote the
paper, chose LP4FM as the venue and made both submissions. For the camera-ready she answered
both reviews, found the segmenter error and re-scored all nine legs from their raw
generations, put the equivalence verdicts on unrounded intervals, and added the October
DeepSeek re-run as a tenth leg. She released the public dataset and this repository jointly
with Dico Angelo, and steered the poster's revision around the statistical tests and the
few-shot result.

**Dico Angelo**: the initial burstiness-engine repository, set up in June for the project's
earlier in-generation controller scope, where he built and tested a boundary-FUDGE decoding
controller on GPT-2 medium (activation-steering sweeps, paired-decoding robustness checks),
ran a held-out human-versus-machine discriminativeness test, and wrote the perception-study
protocol and the controller-scope paper drafts; CI checks that keep reported numbers in
sync with the results and the paper build system, which also support this paper; the project
site (https://burstiness-research.metaventionsai.com); the literature synthesis; compute and
API funding; the generation legs for DeepSeek V4 Pro (including the 18 October re-run and the
paraphrase probe), Grok 4.3, Grok 4.5, Gemini 3.1 Pro, Gemma 3 27B, GLM-5.2, Kimi K2.6 and
Claude Opus 4.8; the generation-hygiene audit and uniform normalization; the equivalence,
multiplicity and contamination analyses; the moderation-refusal audit; the table generator;
caught the curly-quote artifact in segmentation entropy in July and re-scored DeepSeek and
Grok (commits c2fc621, 7b1ff70 and ed1575a in the original repository); and presenting the
poster at LP4FM in Sydney (NeurIPS 2026).

Development history: github.com/Dicoangelo/burstiness-engine (private; available on request).
The camera-ready work is in this repository's history.

## License

Code: Apache 2.0 (see LICENSE). Data: the Hugging Face dataset burstiness-engine/sentence-length-dispersion is
released under CC BY 2.5, the licence of the English Wikinews articles it contains; its card carries the attribution.
