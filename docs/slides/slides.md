# Pitch slides (draft, without the Gantt chart)

Text of the three slides built for the kick-off pitch (max. 3 minutes). The editable deck is a Claude Slides artifact; its source files are in `source/`. Download a PowerPoint or PDF from the deck's menu and save it next to this file.

The course asks for 2 slides in total (planning and risks, then the Gantt chart), so these three get merged once the Gantt is added.

## Slide 1: Explore each outlet’s style, then write in it

**Explorer**
- Style fingerprint per outlet: sentence length, quotes, readability
- Sociopolitical map of outlets, based on cited external ratings
- Word shifts: what one outlet over-uses against another on the same topic
- Outlet by topic heatmap of any metric

**Generator**
- Small open model (1B–4B), fine-tuned with LoRA, served locally with Ollama
- Input: outlet, topic and date
- Output: an article in that outlet’s style
- Marks the patterns in the text: loaded wording, hedging, quote and attribution style

**Data:** CC-News articles labeled by outlet, topic and date. Bias is measured against cited external ratings.

## Slide 2: Five stages, from CC-News to the platform

| Stage | What |
|---|---|
| 01 Data | CC-News (stanford-oval/ccnews): 4–6 outlets, deduped, topic-labeled |
| 02 Training | LoRA on a 1B–4B model, conditions as control codes, MLX-LM on Mac |
| 03 Grounding | Retrieval over the corpus for facts; Scrapling fills coverage gaps |
| 04 Evaluation | Outlet classifier, stylometrics, bias against external ratings, claim check |
| 05 Platform | Explorer plus generator that marks patterns in the text |

**Five variants compared:** prompt only, few-shot, one LoRA, LoRA per outlet, LoRA with retrieval.

## Slide 3: Roles, deliverables and risks

**Roles:** Data and literature: [Name] · Fine-tuning: [Name] · Retrieval and platform: [Name] · Evaluation and ethics: [Name]

**Deliverables:** written document with findings · code and implementation · final presentation

| Risk | Mitigation |
|---|---|
| Laptops too slow to train | Small model with QLoRA; Colab for the final run |
| Outlet coverage and licensing in CC-News | Count articles per outlet first; cite as research use; Scrapling fills gaps |
| Hallucinated facts | Retrieval grounding and a claim check |
| Bias claims without ground truth | Cited external outlet ratings |
| Time overrun | Cut the per-outlet LoRAs first |
