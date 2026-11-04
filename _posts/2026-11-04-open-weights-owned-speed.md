---
layout: post
title: "Open weights, and the speed you own"
date: 2026-11-04 00:00:00 +0100
image: /images/same-fire-two-hydrants.jpg
excerpt: "A metered frontier API gives one request 50 to 100 tokens a second. An 8-GPU H200 node running an open-weights model can give the same request about 400."
---

![Cartoon, "Same fire. Two hydrants." At 2 am a building labelled "Tonight's backlog" is on fire. On the left, a blue hydrant labelled "Frontier API, 50–100 tok/s, metered" feeds several trucks and firefighters whose thin streams barely reach the fire; one says "Still rate-limited…". On the right, a magenta hydrant labelled "H200 node, ~400 tok/s, yours, at 2 am" feeds one ladder truck whose firefighter hits the fire with a huge stream and says "Open weights unlocked!"](/images/same-fire-two-hydrants.jpg)

A metered frontier API gives one request 50 to 100 tokens a second. An 8-GPU H200 node running an open-weights model can give the same request about 400.

That speed is yours today. Matching the frontier on every task isn't, yet, and the difference decides what you put on the node.

## The speed you can own today

A metered API hands you one operating point: its stream speed, with a tokens-per-minute cap above it. On your own node you choose the operating point. Point the whole node at a single request and the arithmetic changes.

Take GLM-5.3-Flash: 320B parameters in total, 18B active per token, MIT licence. At FP8 it fits on one 8-GPU H200 node with most of the memory free. Each token reads 18 GB of weights against 38 TB/s of memory bandwidth across the node, so the ceiling sits near 2,100 tokens a second. Real serving stacks reach about a fifth of that for a single request. Call it 400: four to eight times a metered stream, and nothing rate-limits it. Batching flips the trade: many requests share each weight read, so the node's total output rises far above 2,100 tokens a second while each request slows down. The full chain is at the end.

That speed matters wherever one step waits for the last. An agent run of 40 sequential steps, each writing 1,000 tokens, takes under two minutes at 400 tokens a second, and 7 to 13 minutes on a metered stream.

Bigger is not better here, either. On Artificial Analysis Index v4.3, GLM-5.3-Flash scored 42, above Qwen3.8 at 2.4T parameters (40) and DeepSeek V4 Pro at 1.6T (36). Those need three and two nodes respectively at FP8.

## What it costs

An 8-GPU H200 node costs about $370,000. All-in, with four-year depreciation, its share of storage and networking, power, support and operations, I put it at about €280,000 a year.

At frontier list prices, that pays for itself once your workload averages roughly 800 to 5,000 tokens a second around the clock. The range depends on the model and on how much of your input is cached. A node handles that easily, because most of those tokens are input. The real question is whether you have the demand.

That is where the night shift comes in. A nine-to-five team uses 50 of the week's 168 hours, so the hardware would sit idle 70% of the time. Overnight is when the corpus gets reprocessed, the evals rerun and the index rebuilt. That is throughput work: many requests at once, not one fast one. During the day you buy speed for interactive and agentic work; at night you fill the node. Frontier batch APIs halve the price of that overnight work, which doubles the break-even, but it stays within reach.

Against renting the same open model through an API, owning only pays off if you keep the node as busy and as well tuned as a provider does, around the clock — at 60,000 tokens a second or more. In-house deployments rarely manage that. So owning isn't the cheapest way to run open weights. It's the cheapest way to run them on your terms: your data in your racks, and the speed you choose.

## Where open models aren't there yet

Epoch AI measured open weights about three months behind the closed frontier in October 2025, and about four months behind in May 2026. Those are two separate measurements, so treat any trend as an extrapolation. If the lag keeps growing at that rate, an open model matching today's frontier, Claude Opus 5.5 and GPT-6 Astra (both September 2026), arrives in the first half of 2027.

![Line chart: the closed frontier rises faster than open weights because the measured lag grew from three months in October 2025 to four by May 2026. Claude Opus 5.5 and GPT-6 Astra mark the frontier in September 2026; open weights reach that level between February and May 2027. GLM-5.3-Flash fits one eight-GPU node; Qwen3.8 2.4T and DeepSeek V4 Pro need two to three.](/images/open-weights-lag.svg)

*Horizontal distance is the measured quantity; the vertical scale is schematic. The chart answers when, not how much.*

The gap is not spread evenly. Open models are at or near parity on extraction, classification, routing and retrieval Q&A. The frontier keeps its edge on sustained multi-step reasoning and long agentic work.

That is where the harness earns its money. If the scaffold holds the plan and checks each step with code rather than the model's judgment, you have stopped asking the model for the capability you are short of. I have watched a good harness let a much smaller model do work a frontier model was doing unscaffolded, at an order of magnitude less per token.

One caveat: most of these parity claims come from vendor write-ups, not controlled evaluation.

## What I'd do

- Put the parity tasks on your own node first: extraction, classification, routing, retrieval Q&A.
- Keep long agentic work on the frontier for now, behind a harness that checks every step.
- Fill the nights. Schedule re-indexing, evals and reprocessing so the node earns its keep.
- Keep the model replaceable. Pin to MIT or Apache licences, and qualify two model families from different jurisdictions behind one adapter, so a licence change or an export rule doesn't strand you.
- Measure before you buy: blind-labelled evals on your own documents.

## The arithmetic

**Speed for a single request**

| Step | Value | Source |
| --- | --- | --- |
| Active parameters per token | 18B | GLM-5.3-Flash specs |
| Bytes read per decode step at FP8 | 18 GB | 18B × 1 byte |
| HBM3e bandwidth per GPU, H200 | 4.8 TB/s | NVIDIA spec sheet |
| Aggregate across an 8-GPU node | 38.4 TB/s | 8 × 4.8 |
| Bandwidth ceiling at batch 1 | ~2,100 tok/s | 38.4 ÷ 0.018 |
| Realised fraction at batch 1, TP8 | ~20% | calibrated against dense 70B |
| **Realistic single-stream decode** | **~400 tok/s** | 2,100 × 0.20 |
| Frontier API, per stream | 50–100 tok/s | published median output speeds |
| **Multiple** | **4–8×** | |

The efficiency factor is the one judgement call, and it is calibrated rather than guessed: dense 70B at FP8 on eight H100s has a bandwidth ceiling near 380 tok/s and measures 60–80 at batch 1, which is about a fifth. Applying the same fraction to the mixture-of-experts arithmetic is the conservative move; expert routing and per-layer all-reduce are why it is not higher.

Even if that factor is off, the structural point holds: on a metered stream you are handed one operating point and a rate limit, and on your own node you choose it.

**Cost break-even for one node**

Throughput the node must average, around the clock, to cost the same as the API at list prices:

| Compared with | Heavy caching | Light caching |
| --- | --- | --- |
| GPT-6 Astra ($10 in / $1 cached / $50 out) | ~1,900 tok/s | ~800 tok/s |
| Claude Opus 5.5 ($4 / $0.20 / $20) | ~5,200 tok/s | ~2,100 tok/s |
| GLM-5.3-Flash via Z.ai ($0.15 / $0.03 / $0.50) | ~131,000 tok/s | ~62,000 tok/s |

Assumptions: one node at about €282,000 a year all-in, or $315,000 at €1 = $1.117. Heavy caching means 19% new input, 76% cached input and 5% output; light caching means 70% new input, 20% cached input and 10% output. Prices are per million tokens, list, as of November 2026. Batch APIs halve the frontier prices, which doubles the first two rows.

## Sources

- Epoch AI, [Open-weight models lag state-of-the-art by around 3 months on average](https://epoch.ai/data-insights/open-weights-vs-closed-weights-models) (October 2025)
- Epoch AI, [open versus closed capability gap: four months on average since January 2026](https://epoch.ai/data-insights/open-closed-eci-gap) (May 2026)
- [Artificial Analysis Intelligence Index v4.3](https://artificialanalysis.ai/articles/artificial-analysis-intelligence-index-v4-3): scores for GLM-5.3-Flash, Qwen3.8 2.4T and DeepSeek V4 Pro
- [GLM-5.3-Flash: 320B total, 18B active, MIT](https://featherless.ai/blog/glm-5-3-flash-is-live-on-featherless)
- [DeepSeek V4 Pro: 1.6T total, 49B active](https://framia.converge.ai/page/en-US/news/deepseek-v4-parameters)
- [Claude Opus 5.5 release, 22 September 2026](https://emergent.sh/news/opus-5-5-release-date) and [GPT-6 Astra launch, 3 September 2026](https://datanorth.ai/news/openai-launches-gpt-6-astra)
- Prices: [Claude API pricing](https://platform.claude.com/docs/en/about-claude/pricing), [GPT-6 Astra model page](https://developers.openai.com/docs/models/gpt-6-astra), [GLM-5.3-Flash pricing](https://www.eesel.ai/blog/glm-5-3-flash-pricing)
- OEM price guides, 2026: 8-GPU H200 HGX node at roughly $370k integrated
- [NVIDIA H200](https://www.nvidia.com/en-us/data-center/h200/): 4.8 TB/s HBM3e per GPU
