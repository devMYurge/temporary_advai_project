# News generation conditioned on outlet, topic and date

Advanced AI group project (IE University, October 2026). Project I from the course brief: fine-tuning a GPT-style model for automated news generation.

## Goal

A platform with two parts.

1. **Explorer.** Visualizes what each newspaper does: language patterns, sociopolitical lean and bias, per outlet and per topic.
2. **Generator.** A small open model, fine-tuned with LoRA, that writes an article from three conditions (outlet, topic, date) and highlights those patterns in the text it produces.

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

## Layout

```
README.md
docs/
  research.md     State of the art, data, model, experiments, evaluation, sources
```

Planned: `data/` (git-ignored), `src/pipeline/`, `src/train/`, `src/eval/`, `app/`.

The course PDFs in this folder are not tracked (see `.gitignore`).
