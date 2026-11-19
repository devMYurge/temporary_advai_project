# News generation conditioned on outlet, topic and date

Advanced AI group project (IE University, October 2026). Project I from the course brief: fine-tuning a GPT-style model for automated news generation.

## Goal

A platform with two parts.

1. **Explorer.** Visualizes what each newspaper does: language patterns, sociopolitical lean and bias, per outlet and per topic.
2. **Generator.** A small open model, fine-tuned with LoRA, that writes an article from three conditions (outlet, topic, date) and marks those patterns in the text it produces.

## Deliverables (from the course brief)

1. Written document with codebase, methodology and findings.
2. Code with the implementation.
3. Presentation of outcomes and learnings.

The kick-off session also asks for a 2-slide planning document and a pitch of at most 3 minutes: slide 1 covers planning and risks, slide 2 holds the Gantt chart.

## Decisions so far

| Topic | Decision |
|---|---|
| Data | CC-News, via the `stanford-oval/ccnews` dataset on Hugging Face |
| Fine-tuning method | LoRA / QLoRA on a 1B–4B open model |
| Training tool | `mlx_lm.lora` on Apple Silicon |
| Serving | Ollama, after exporting the fused model |
| Scraping | Scrapling, only to fill gaps CC-News does not cover |

## Open decisions

- Which 4–6 outlets (distinct style and politics, enough volume in CC-News).
- Time window for the corpus.
- Base model family. The GGUF export in MLX-LM supports Llama, Mistral and Mixtral architectures only.
- Group size and roles.

## Setup

Everyone uses the same conda environment, `advai-news`, defined in `environment.yml` (Python 3.12, packages from conda-forge, pinned versions).

```bash
# 1. Install Anaconda or Miniconda (once per machine)

# 2. Create the environment
conda env create -f environment.yml

# 3. Activate it before running anything
conda activate advai-news
# notebooks (pick the advai-news kernel)
jupyter lab
# scripts
python src/pipeline/...
```

Rules:

- Add a dependency by editing `environment.yml` (pinned version), then run `conda env update -f environment.yml --prune`. Do not `pip install` or `conda install` by hand, or the environments drift apart.
- After pulling a change to `environment.yml`, run the same update command.
- `mlx-lm` (training) installs only on Apple Silicon Macs. Everything else works on any OS. Teammates on other machines can do data, EDA and evaluation, and the training runs on a Mac.
- Data goes in `data/` (git-ignored). The Hugging Face dataset downloads into the HF cache on first load.

## Layout

```
README.md
environment.yml      Shared conda environment (advai-news)
notebooks/           EDA and exploration
docs/
  research.md          State of the art, data, model, experiments, evaluation, sources
  brief/               Course instructions (group project brief, kick-off deck)
  slides/              Pitch slides: slides.md (text) and source/ (editable deck)
```

Planned: `data/` (git-ignored), `src/pipeline/`, `src/train/`, `src/eval/`, `app/`.
