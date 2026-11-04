---
layout: post
title: "Open weights, and the speed you own"
date: 2026-11-04 00:00:00 +0100
image: /images/same-fire-two-hydrants.jpg
---

![Cartoon, "Same fire. Two hydrants." At 2 am a building labelled "Tonight's backlog" is on fire. On the left, a blue hydrant labelled "Frontier API, 50–100 tok/s, metered" feeds several trucks and firefighters whose thin streams barely reach the fire; one says "Still rate-limited…". On the right, a magenta hydrant labelled "H200 node, ~400 tok/s, yours, at 2 am" feeds one ladder truck whose firefighter hits the fire with a huge stream and says "Open weights unlocked!"](/images/same-fire-two-hydrants.jpg)

Imagine running your Opus 5.5 skills locally. Five times faster, on hardware you already idle 70% of the week.

Not this year. Epoch AI puts open weights about four months behind the closed frontier, and the lag is widening: three months in October 2025, four by May 2026. Extend it at its own measured rate and the open model matching today's Opus 5.5 lands around the middle of 2027.

The gap is also not spread evenly. Open models are at or near parity on extraction, classification, routing and retrieval Q&A. The frontier keeps its edge on sustained multi-step reasoning and long agentic work. That is where the harness earns its money. If the scaffold holds the plan and checks each step with code rather than the model's judgment, you have stopped asking the model for the capability you are short of. I have watched a good harness let a much smaller model do work a frontier model was doing unscaffolded, and the cost difference is not 20%.

The hardware is unremarkable. An 8-GPU H200 node costs about $370,000 and holds a 320B model at FP8 with most of its memory free. Bigger is not better: that 320B scored above both a 2.4T and a 1.6T open release, and those need three to five nodes each.

What nobody prices is speed. A metered API hands you one operating point: roughly 50 to 100 output tokens a second on a stream, with a tokens-per-minute cap above it. Point a whole node at a single request and the arithmetic changes. A 320B mixture-of-experts reads 18B active parameters per token, 18 GB at FP8, against 38 TB/s across eight H200s, so the bandwidth ceiling sits near 2,100 tokens a second. Real stacks reach about a fifth of that at batch one, the same factor that reproduces measured decode rates for dense 70B models. Call it 400. Four to eight times a metered stream, and nothing rate-limits it.

That is the trade an API will not make for you: fewer concurrent requests, each sharded across more GPUs. And a nine-to-five company has the capacity to spend, because 50 working hours out of 168 leaves the fleet idle 70% of the week. Overnight is when the corpus gets reprocessed, the evals rerun, the index rebuilt. Those hours are already bought, at a speed you cannot rent.

One caveat: most task-class parity claims come from vendor write-ups, not controlled evaluation. Measure on your own documents, blind-labelled, before you commit capital.

The model is the replaceable part. Pin to MIT or Apache, qualify two families from different jurisdictions behind one adapter.

![Line chart: the closed frontier rises faster than open weights because the measured lag grew from three months in October 2025 to four by May 2026. Claude Opus 5.5 ships October 2026; open weights reach that level between March and June 2027. A 320B-class model fits one eight-GPU node.](/images/open-weights-lag.svg)

*Horizontal distance is the measured quantity; the vertical scale is schematic. The chart answers when, not how much.*

## Where "five times faster" comes from

The full chain behind the headline multiple:

| Step | Value | Source |
| --- | --- | --- |
| Active parameters per token | 18B | 320B-class sparse MoE, model config |
| Bytes read per decode step at FP8 | 18 GB | 18B × 1 byte |
| HBM3e bandwidth per GPU, H200 | 4.8 TB/s | NVIDIA spec sheet |
| Aggregate across an 8-GPU node | 38.4 TB/s | 8 × 4.8 |
| Bandwidth ceiling at batch 1 | ~2,100 tok/s | 38.4 ÷ 0.018 |
| Realised fraction at batch 1, TP8 | ~20% | calibrated against dense 70B |
| **Realistic single-stream decode** | **~400 tok/s** | 2,100 × 0.20 |
| Frontier API, per stream | 50–100 tok/s | published median output speeds |
| **Multiple** | **4–8×** | stated in the post as five |

The efficiency factor is the one judgement call, and it is calibrated rather than guessed: dense 70B at FP8 on eight H100s has a bandwidth ceiling near 380 tok/s and measures 60–80 at batch 1, which is about a fifth. Applying the same fraction to the MoE arithmetic is the conservative move, and expert routing plus per-layer all-reduce are why it is not higher.

Even if that factor is off, the structural point holds: on a metered stream you are handed one operating point and a rate limit, and on your own node you choose the operating point.

## Sources

- Epoch AI, [Open-weight models lag state-of-the-art by around 3 months on average](https://epoch.ai/data-insights/open-weights-vs-closed-weights-models) (October 2025)
- Epoch AI, [open versus closed capability gap: four months on average since January 2026](https://epoch.ai/data-insights/open-closed-eci-gap) (May 2026)
- Hugging Face, *State of Open Models, Summer 2026*: licence mix and release sizes
- [Artificial Analysis Intelligence Index v4.3](https://artificialanalysis.ai/articles/artificial-analysis-intelligence-index-v4-3): the one version on which the 320B, the 2.4T and the 1.6T compare
- OEM price guides, 2026: 8-GPU H200 HGX node at roughly $370k integrated
- [NVIDIA H200](https://www.nvidia.com/en-us/data-center/h200/): 4.8 TB/s HBM3e per GPU
