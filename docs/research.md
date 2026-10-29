# Research notes

Collected 2026-10-29. Items marked **unverified** were not confirmed against the source.

## 1. The task

Generate a news article from three conditions: outlet, topic, date. The output should carry the outlet's style, audience and bias, and the platform should make those patterns visible. The course brief asks for four phases: data collection and literature review, fine-tuning, prompt engineering or retrieval, and ethics.

## 2. State of the art for conditioned and style-controlled generation

| Technique | What it is | Status for us |
|---|---|---|
| Control codes in the prompt | Put `[outlet] [topic] [date]` in the prompt and fine-tune on that format. See Lilian Weng's survey of controllable text generation. | Main method |
| LoRA / QLoRA style tuning | Parameter-efficient fine-tuning to shift writing style. StyleTunedLM (CMU) applies LoRA to LLaMA-2 7B. Only the method was readable, not the results. | Main method |
| One LoRA per outlet | Train a small adapter per newspaper and swap them. ORAL studies prompting large pools of LoRAs. | Comparison experiment |
| Inference-time style control | Steer the output distribution at decoding time. "Limits of n-gram Style Control via Logit-Space Injection" reports real limits. | Contrast case |
| Retrieval grounding | Supply source articles at generation time so facts come from text, not weights. | Needed for factuality and for the date condition |
| Political bias measurement | Yoo and Shin (EMNLP 2025) analyzed 10,850 articles from seven LLMs. They found persistent left-leaning bias, rare neutral output, and flipped quotations in 33–38% of quotes. Larger models were not more neutral. They released PublicViews and code. | Adapt their protocol |

No paper found targets "generate by topic, date and outlet" directly.

## 3. Data: CC-News

Common Crawl is a nonprofit that publishes a free crawl of the public web. CC-News is its news-only collection, released as gzipped WARC files under `crawl-data/CC-NEWS/` (2016 to present).

Use the Parquet version instead of raw WARC:

- `stanford-oval/ccnews` on Hugging Face. About 461M rows, subsets 2016–2024. Columns: `requested_url`, `responded_url`, `plain_text`, `published_date`, `crawl_date`, `title`, `tags`, `categories`, `author`, `image_url`, `sitename`, `publisher`, `language`, `language_score`, `warc_path`. License not stated on the page.
- `vblagoje/cc_news` as a small fallback. 708,241 English articles, Jan 2017 to Dec 2019, extracted with news-please. License listed as unknown.

Cautions:

- Coverage varies by outlet. Some sites block the crawler, and paywalled outlets give only free text. Count articles per outlet per year before choosing outlets.
- Archived content may be truncated (Common Crawl erratum, 1 MiB before March 2025, 5 MiB after; unclear whether it applies to CC-News).
- No topic label. Derive it from `categories`, `tags` or the URL path.
- Copyright stays with the outlets. Cite as research use.
- Exact `sitename` values and a working filter query are **unverified**. Test on a small stream.

### Scrapling

Python scraping library (HTTP fetcher, browser fetcher, adaptive parsing, spiders with JSONL export). Install with `pip install "scrapling[fetchers]"` then `scrapling install`. Use it only to fill outlets or dates CC-News lacks, and set `robots_txt_obey`. Its anti-bot bypass claims are the project's own and were not verified. Bypassing protection on paywalled sites weakens the ethics section.

### Pipeline

1. Stream the dataset, filter by `sitename`, English, date window, minimum length.
2. Clean boilerplate (subscribe prompts, cookie text, bylines).
3. Dedupe with MinHash. Wire copy (AP, Reuters) appears across outlets and blurs the style signal.
4. Map `categories` or URL path to 4–5 shared topics.
5. Sample balanced by outlet × topic. About 5 × 4 × 1–2k articles is 20–40k examples.
6. Split by time window and by event, not by random row, so near-duplicates do not leak into test.

## 4. Training

### LoRA

LoRA freezes the base model and learns a low-rank update ΔW = B·A for selected weight matrices. With rank r = 16 on a 4096×4096 matrix that is about 131k trainable numbers instead of 16.8M. Result: a small adapter that can be swapped or fused. It is good for style and tone and weak at adding new facts. QLoRA runs LoRA on a quantized base model for lower memory.

### Format

Chat-style JSONL, one example per line:

```json
{"messages":[
 {"role":"user","content":"Outlet: The Guardian | Topic: politics | Date: 2019-03-12 | Headline: ..."},
 {"role":"assistant","content":"<article text>"}]}
```

Train with `--mask-prompt` so loss covers only the article.

### Tooling (MLX-LM)

- Install: `pip install "mlx-lm[train]"`.
- Train: `mlx_lm.lora --model <path> --train --data <dir> --iters 600`. A quantized model means QLoRA. Types: `lora`, `dora`, `full`.
- Evaluate: `mlx_lm.lora` with `--adapter-path --data --test` (test perplexity).
- Generate: `mlx_lm.generate --model --adapter-path --prompt`.
- Memory levers: `--batch-size`, `--grad-accumulation-steps`, `--num-layers`, `--grad-checkpoint`, shorter examples.
- Fuse: `mlx_lm.fuse --model <path>`. `--export-gguf` works only for Mistral, Mixtral and Llama-style models in fp16.
- MLX-LM docs do not cover Ollama import. Check Ollama's documentation.
- Starting values (general practice, not tuned): rank 16, learning rate 1e-4 to 2e-4, 1–2 epochs, sequence length 1024.

### Model choice

Pick a 1B–4B open model. Prefer Apache 2.0 licenses. Because of the GGUF export limit, a Llama-architecture model (Llama 3.2 1B or 3B) gives the clean path to Ollama. Debug on 1B first. Use free Colab for the final run if laptops cannot handle it. Specific current model names and sizes are **unverified**; check before committing.

Unsloth needs an NVIDIA GPU, so it does not apply on Macs.

## 5. Experiments

| Variant | Tests |
|---|---|
| A. Base model, prompt only | Baseline: "write like The Guardian" |
| B. Base model, few-shot | Retrieved real examples from the outlet in the prompt |
| C. One LoRA with control codes | Main method |
| D. One LoRA per outlet | Adapter-swapping alternative |
| E. C plus retrieval grounding | Facts from retrieved articles |

## 6. Evaluation

- **Style transfer.** Train an outlet classifier on real articles (TF-IDF plus logistic regression, or a small BERT). Run it on generated text. Accuracy against the requested outlet is the style score.
- **Stylometrics.** Sentence length, headline length, quote rate, readability, first-person use. Compare real and generated distributions per outlet.
- **Fluency.** Perplexity on held-out articles per outlet.
- **Bias.** Same topic and date across all outlets, then measure political leaning and framing. Ground truth for outlet leaning must come from an external rating source (AllSides or Media Bias/Fact Check; cite it). Adapt the Yoo and Shin protocol, including the flipped-quotation check.
- **Factuality.** Split generated text into claims and check each against retrieved sources with an NLI model.
- **Date leakage.** Request a date before a known event and test whether the model mentions it.
- **Human rubric.** 20–30 articles, rated 1–5 for style match, plausibility and bias, by at least two raters.

## 7. Platform ideas (proposals, not decisions)

Explorer views:

- Style fingerprint per outlet (stylometric profile).
- Sociopolitical map: outlets placed on a lean axis using the external ratings plus measured language signals.
- Word-shift view: words over-used by one outlet versus another on the same topic (log-odds ratio).
- Outlet × topic heatmap of a chosen metric.

Generator view:

- Output text with spans highlighted by pattern type (loaded wording, hedging, attribution style, quote use), linked to the explorer view for that outlet.
- Side-by-side generation of the same topic and date for several outlets.

## 8. Ethics

- State clearly that the platform measures and shows bias; it does not endorse it.
- Label all generated articles as AI-generated.
- Add a fact-check step against retrieved sources.
- Document data provenance and license status for every source.
- Risk: a model that writes convincingly in a named outlet's voice can be used to fabricate attributed articles. Do not present outputs as real articles from that outlet.

## 9. Sources

- Scrapling: https://github.com/D4Vinci/Scrapling
- stanford-oval/ccnews: https://huggingface.co/datasets/stanford-oval/ccnews
- vblagoje/cc_news: https://huggingface.co/datasets/vblagoje/cc_news
- Common Crawl CC-News announcement: https://commoncrawl.org/blog/news-dataset-available
- MLX-LM LoRA documentation: https://github.com/ml-explore/mlx-lm/blob/main/mlx_lm/LORA.md
- Fair or Framed? Political bias in news articles generated by LLMs (Yoo and Shin, EMNLP 2025): https://preview.aclanthology.org/setup/2025.emnlp-main.856
- Controllable neural text generation (Lilian Weng): https://lilianweng.github.io/posts/2021-01-02-controllable-text-generation/
- ORAL: https://aclanthology.org/2025.findings-emnlp.71.pdf
- Limits of n-gram style control via logit-space injection: https://arxiv.org/html/2601.16224
- StyleTunedLM review: https://www.themoonlight.io/en/review/customizing-large-language-model-generation-style-using-parameter-efficient-finetuning
- Fine-tune LLM locally guide: https://toolhalla.ai/blog/fine-tune-llm-locally-guide-2026
