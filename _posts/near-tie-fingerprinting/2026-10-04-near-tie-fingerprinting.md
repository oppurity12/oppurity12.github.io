---
title: "Language Model Fingerprinting Requires Rethinking Watermark Teachers"
subtitle: "We rethink whether text watermarks are suitable distillation teachers for model fingerprinting. Aggregation across responses permits sparser signals; near-tie restriction places the bias on plausible tokens, improving generation quality at comparable detectability."
description: "We revisit the generation-quality cost of watermark-based model fingerprinting and improve the detection–quality trade-off by restricting the teacher’s bias to near-tie tokens."
post_type: Research
venue: Preprint
authors:
  - name: Jeongyeon Hwang
    affil: 1
    url: /
  - name: Anshul Nasery
    affil: 2
    url: https://anshuln2.github.io/
  - name: Sewoong Oh
    affil: 2
    url: https://homes.cs.washington.edu/~sewoong/
  - name: Jungseul Ok
    affil: 1
    url: https://sites.google.com/view/jungseulok
affiliations:
  - POSTECH
  - University of Washington
links:
  - label: Paper
    icon: fas fa-file-pdf
    url: https://arxiv.org/abs/2610.04169
  - label: Code
    icon: fab fa-github
    url: https://github.com/ml-postech/near-tie-fingerprinting
tldr:
  - "**Prior utility evaluation understates quality degradation** in the fingerprint domain. On Llama-3.1-8B, benchmark accuracy changes only modestly (0.64 → 0.61), while perplexity on open-ended French generation rises **4.63 → 28.27** and an LLM-judge score falls **7.76 → 3.73**."
  - "**We rethink watermark teachers for model fingerprinting**, where aggregating responses permits sparser signals. Our analysis shows that placement matters even at matched teacher strength, motivating **near-tie restriction**: bias only green tokens close to the base model’s top prediction."
  - "**Near-tie improves the detection–quality trade-off**: on Llama-3.1-8B, at comparable worst-case detectability, the PPL ratio drops **6.09 → 1.04** and the judge score improves **3.52 → 7.40** (base 7.51). Improvements hold across four models and query budgets, and near-tie also improves existing watermarking schemes when combined with them."
bibtex: |
  @article{hwang2026fingerprinting,
    title   = {Language Model Fingerprinting Requires Rethinking Watermark Teachers},
    author  = {Hwang, Jeongyeon and Nasery, Anshul and Oh, Sewoong and Ok, Jungseul},
    journal = {arXiv preprint arXiv:2610.04169},
    year    = {2026}
  }
---

<figure class="post-figure post-figure--hero">
  <a href="/images/blog/near-tie-fingerprinting/overview.png" aria-label="Open the overview figure at full resolution"><img src="/images/blog/near-tie-fingerprinting/overview.png" alt="Overview: watermark-based fingerprinting distills a text watermark into model weights; the prior KGW teacher biases every green token, near-tie biases only green tokens near the top-1 logit"></a>
  <figcaption><b>Figure 1.</b> Top: a text watermark is distilled into model weights on a fingerprint domain. Bottom: the prior KGW teacher biases every green token; near-tie restricts the bias to green tokens close to the top-1 logit.</figcaption>
</figure>

<details class="post-toc" markdown="1">
<summary>Contents</summary>

* TOC
{:toc}

</details>

## Model fingerprinting for open-weight LLMs

As more capable LLMs are released with open weights, they can be deployed behind black-box APIs without honoring their licenses, raising concerns about model ownership and unauthorized use. A healthy open-weight ecosystem therefore needs reliable ownership verification that works from API responses alone. *Black-box fingerprinting* serves this purpose: it embeds a recognizable signal in the model before release so the owner can later test for it.

We focus on watermark-based fingerprinting ([Gloaguen et al., 2026](https://arxiv.org/abs/2505.16723)), which encodes the signal as a statistical pattern in responses from a secret *fingerprint domain*, such as French. Compared with backdoor fingerprints based on secret fixed trigger–response pairs, it produces more natural-looking outputs and better withstands deployment transformations, e.g., pruning, quantization, and fine-tuning.

## How watermark-based fingerprinting works

Watermark-based fingerprinting builds on [KGW](https://arxiv.org/abs/2301.10226), a widely used LLM watermarking scheme. Using a secret key and the previous h tokens, KGW pseudorandomly splits the vocabulary into *green* and *red* lists. It adds δ to green-token logits, making these tokens more likely to appear.

For a response $$x$$ with $$L$$ tokens, the detector compares the observed green count $$N_G(x)$$ with the null expectation $$\gamma L$$, where $$\gamma$$ is the green-list fraction:

$$
z(x) = \frac{N_G(x) - \gamma L}{\sqrt{\gamma(1-\gamma)L}}.
$$

A one-sided test declares the text watermarked when $$z(x) > \rho$$.

For open-weight models, the owner cannot enforce this decoding rule after release. Fingerprinting therefore distills the KGW-biased teacher’s distribution into the model’s weights using data from a secret semantic domain (e.g., French), enabling the signal under ordinary decoding.

To verify model ownership, the owner queries a suspect model with prompts from this domain and aggregates its responses. The same test is applied after deduplicating context–token pairs, with green-list membership checked in each token’s original context.

<details class="post-more" markdown="1">
<summary>Detection threshold and deployment changes</summary>

Our experiments use h = 1 and γ = 0.25. The detection threshold ρ = 4 corresponds to a nominal one-sided false-positive rate of approximately 3 × 10⁻⁵. We also test changes to sampling, system prompts, and model weights (quantization, pruning, and fine-tuning). The *worst-case z-score* is the lowest mean score across these transformations, with each mean computed over five seeds.

</details>

<figure class="post-figure post-figure--svg">
  <div class="post-figure__scroll" tabindex="0" role="group" aria-label="Scrollable figure">
  {% include blog/near-tie-fingerprinting/fig-semantic.svg %}
  </div>
  <figcaption><b>Figure 2.</b> Ownership verification with French as the fingerprint domain. The owner queries a suspect model in French and aggregates evidence across its responses to detect the fingerprint.</figcaption>
</figure>

## Prior evaluation understates the quality cost
{: #fingerprinting-degrades-generation-quality-in-the-fingerprint-domain }

Legitimate users also rely on the model within its fingerprint domain, so fingerprinting should preserve generation quality there. The prior evaluation reports accuracy on the French Benchmark and a metric labeled “PPL” that is actually mean token entropy. These metrics understate the degradation we observe in open-ended generation.

We reproduce the Llama-3.1-8B-Instruct setting (KGW, δ = 4) on 500 WritingPrompts prompts translated into French. Qwen-2.5-32B measures perplexity as an external reference model, and GPT-5 scores grammar, fluency, and coherence on a 1–10 scale. Figure 3 reports absolute perplexity; subsequent results use the ratio to the base model’s perplexity.

<figure class="post-figure">
  {% include blog/near-tie-fingerprinting/fig-metrics.html %}
  <figcaption><b>Figure 3.</b> Utility re-evaluation. A 3-point drop in benchmark accuracy accompanies a much larger increase in reference-model perplexity and a decline in judged French generation quality.</figcaption>
</figure>

**Reducing the bias improves quality at the expense of detectability.** On Llama-3.1-8B, δ = 2 still yields a PPL ratio of 2.16, while δ = 1 falls below the worst-case detection threshold. Lowering δ alone does not recover near-base quality with reliable detection in this sweep.

<figure class="post-figure post-figure--narrow">
  <a href="/images/blog/near-tie-fingerprinting/delta-sweep.png" aria-label="Open the bias sweep at full resolution"><img src="/images/blog/near-tie-fingerprinting/delta-sweep.png" alt="KGW bias sweep on Llama-3.1-8B: delta 2 improves quality over delta 4, delta 1 falls below the detection threshold" loading="lazy"></a>
  <figcaption><b>Figure 4.</b> Sweeping the KGW bias δ on Llama-3.1-8B. x: PPL ratio (lower is better); y: worst-case z-score (higher is better).</figcaption>
</figure>

## Rethinking watermark teachers
{: #fingerprint-verification-allows-sparser-signals }

Prior work uses a decoding-time text watermark as the distillation teacher.

### Aggregation reduces the signal needed per response

Text watermarks are typically designed to remain detectable from an individual output, even when it is short or has been modified. In contrast, model fingerprinting allows aggregating evidence across responses, so each response can carry a weaker signal.

<figure class="post-figure post-figure--svg">
  <div class="post-figure__scroll" tabindex="0" role="group" aria-label="Scrollable figure">
  {% include blog/near-tie-fingerprinting/fig-regimes.svg %}
  </div>
  <figcaption><b>Figure 5.</b> Text watermark detection typically relies on an individual output. Fingerprint verification aggregates evidence across responses within the query budget, making sparser teacher signals viable.</figcaption>
</figure>

### Where should the sparse signal be placed?
{: #placement-matters-even-at-matched-teacher-strength }

We favor green tokens that the base model already considers plausible. We analyze this choice through token surprisal: less probable tokens have higher surprisal.

To separate placement from teacher strength, we fix the base distribution and bias δ and compare green subsets with equal base probability mass. The resulting teachers have identical green-token probability and KL divergence from the base model. Yet the subset with lower average surprisal produces a smaller shift in expected base-model surprisal (Proposition 5.1).

<details class="post-more" markdown="1">
<summary>Proposition 5.1: conditions and equations</summary>

At decoding step $$t$$, let $$Q_t$$ be the base distribution and $$G_t$$ the green list. For a subset $$S \subseteq G_t$$, $$\widetilde Q_t^S$$ denotes the teacher obtained by adding bias $$\delta$$ to the logits of tokens in $$S$$.

The base-model surprisal of a token and the probability-weighted average within $$S$$ are

$$
\begin{aligned}
r_t(v) &= -\log Q_t(v), \\
\bar r_t(S) &= \mathbb{E}_{v\sim Q_t(\cdot\mid S)}[r_t(v)].
\end{aligned}
$$

The teacher’s *surprisal shift* is the change in expected base-model surprisal:

$$
\begin{aligned}
\Delta_t^{\mathrm{surp}}(\widetilde Q_t^S; Q_t)
&= \mathbb{E}_{v\sim\widetilde Q_t^S}[r_t(v)] \\
&\quad - \mathbb{E}_{v\sim Q_t}[r_t(v)].
\end{aligned}
$$

**Proposition 5.1.** Suppose $$S_1,S_2\subseteq G_t$$ satisfy $$Q_t(S_1)=Q_t(S_2)>0$$ and receive the same bias $$\delta>0$$. Their teachers have identical green-token probability and KL divergence:

$$
\begin{aligned}
\widetilde Q_t^{S_1}(G_t) &= \widetilde Q_t^{S_2}(G_t), \\
\mathrm{KL}(\widetilde Q_t^{S_1}\Vert Q_t)
&= \mathrm{KL}(\widetilde Q_t^{S_2}\Vert Q_t).
\end{aligned}
$$

Their surprisal shifts, however, are ordered by the subsets’ average surprisal:

$$
\begin{gathered}
\Delta_t^{\mathrm{surp}}(\widetilde Q_t^{S_1}; Q_t)
\le \Delta_t^{\mathrm{surp}}(\widetilde Q_t^{S_2}; Q_t) \\
\Longleftrightarrow\quad \bar r_t(S_1)\le\bar r_t(S_2).
\end{gathered}
$$

Thus, at matched strength, biasing the subset with lower average surprisal produces a smaller surprisal shift.

</details>

This motivates **near-tie restriction**: biasing only green tokens close to the base model’s top prediction.

<figure class="post-figure post-figure--svg">
  <div class="post-figure__scroll" tabindex="0" role="group" aria-label="Scrollable figure">
  {% include blog/near-tie-fingerprinting/fig-placement.svg %}
  </div>
  <figcaption><b>Figure 6.</b> Toy decoding step (δ = log 3). Both teachers boost green tokens holding 33% of the base probability, so green-token probability and KL match; only the plausibility of the boosted tokens differs.</figcaption>
</figure>

## Near-tie restriction
{: #near-tie-restriction }

We measure closeness to the top prediction by the logit gap, which is also the difference in surprisal.

Let $$\ell_t$$ be the base model’s logits and $$v_t^\star$$ its top-1 token. Because the softmax normalizer cancels, a token’s surprisal $$r_t(v) = -\log Q_t(v)$$ under the base distribution $$Q_t$$ exceeds that of the top-1 by exactly its logit gap:

$$
r_t(v) - r_t(v_t^\star) = \ell_t(v_t^\star) - \ell_t(v).
$$

The teacher applies the bias only to green tokens whose gap is below τ:

$$
S_t^{\mathrm{NT}}(\tau) = \{v \in G_t : \ell_t(v_t^\star) - \ell_t(v) < \tau\}.
$$

Applying this restriction to KGW gives **KGW-NT**. We set τ to the q-th percentile of top-1–top-2 logit gaps on the fingerprint training data. Smaller q gives a sparser teacher, and τ → ∞ recovers KGW.

{% include blog/near-tie-fingerprinting/demo.html %}

After distillation, near-tie also yields lower PPL ratios than random green subsets matched in base probability at every position:

| Calibration percentile q | Random placement: PPL ratio ↓ | Near-tie: PPL ratio ↓ |
|:--|:--:|:--:|
| 30 | 1.11 ± 0.04 | **1.02** |
| 50 | 1.25 ± 0.02 | **1.04** |

Llama-3.1-8B, δ = 4; three random placements per setting.
{: .nt-source-note}

**The detector selects positions using the base model.** The owner re-scores each response using their own model, keeps positions whose top-1–top-2 gap is below τ, and computes the green-token z-score after deduplication. Selection does not use the secret key or green list, preserving the null green-token rate γ. We use the same detection threshold.

<details class="post-more" markdown="1">
<summary>Tokens vs. positions in the detector</summary>

Teacher bias and detector selection are distinct: a green top-1 token receives the bias even when the runner-up lies outside τ, but the detector excludes such positions because selecting them by the top-1 token’s color would depend on the secret key. Position selection and the removal of repeated token pairs are both key-independent, which preserves the null rate; Appendix D of the paper analyzes the null variance and reports an empirical calibration check.

</details>

<figure class="post-figure post-figure--medium">
  <a href="/images/blog/near-tie-fingerprinting/token-budget.png" aria-label="Open the token-budget plot at full resolution"><img src="/images/blog/near-tie-fingerprinting/token-budget.png" alt="The q = 10 and q = 30 near-tie curves are below z = 4 at 256 scored tokens and cross it at larger token budgets" loading="lazy"></a>
  <figcaption><b>Figure 7.</b> Detection evidence vs. number of scored tokens on Llama-3.1-8B. At 256 scored tokens, the q = 10 and q = 30 near-tie configurations fall below z = 4; both cross the threshold as more tokens are scored.</figcaption>
</figure>

## Results

<div class="post-callout post-setup" markdown="1">
- **Models.** Llama-3.2-3B, Qwen-2.5-3B, Llama-3.1-8B, Gemma-2-9B (instruct); the original French setup. Verification uses 1,000 French Alpaca queries with responses capped at 200 tokens.
- **Baselines.** KGW · SWEET (bias only at high-entropy positions) · MorphMark (adaptive bias strength) · **KGW-TK** (bias green tokens within the top-*k*; the rank-based control).
- **Metrics.** Quality: PPL ratio and LLM-judge drop on French WritingPrompts. Detectability: worst-case z-score over {sampling temperatures, system prompts, FP8/INT4 quantization, 50% pruning, French fine-tuning}, each averaged over five seeds.
</div>

### Result 1: near-base quality at comparable detectability

| Teacher (Llama-3.1-8B) | PPL ratio ↓ | Judge score ↑ | Worst-case z-score ↑ |
|:--|--:|--:|--:|
| KGW, δ = 4 | 6.09 | 3.52 | 10.72 |
| KGW, δ = 2 | 2.16 | 6.40 | 8.05 |
| **KGW-NT, q = 50, δ = 4** | **1.04** | **7.40** | **10.05** |
{: .nt-results-table }

Tables 4–5 of the paper (base judge 7.51, KGW δ = 4 judge 3.52). Figure 3 uses Table 1 (base 7.76, KGW 3.73), which the paper reports separately.
{: .nt-source-note}

<figure class="post-figure post-figure--panels post-figure--three-panels post-figure--wide">
  <div class="post-figure__scroll" tabindex="0" role="group" aria-label="Scrollable figure">
  <a href="/images/blog/near-tie-fingerprinting/frontier-ppl-judge.png" aria-label="Open detection–quality plots at full resolution"><img src="/images/blog/near-tie-fingerprinting/frontier-ppl-judge.png" alt="KGW-NT shifts the detection–quality frontier toward lower PPL ratios on Qwen-2.5-3B and Llama-3.1-8B, and toward a smaller LLM-judge drop on Llama-3.1-8B" loading="lazy"></a>
  </div>
  <figcaption><b>Figure 8.</b> Detection–quality frontiers. Left and middle: PPL ratio on Qwen-2.5-3B and Llama-3.1-8B. Right: LLM-judge drop ΔJ = J<sub>base</sub> − J<sub>fp</sub> on Llama-3.1-8B. Each point is a teacher configuration; upper-left is better, and the dashed line marks z = 4. KGW-NT improves the frontier over all evaluated baselines under both quality metrics, including the rank-based KGW-TK control.</figcaption>
</figure>

Across the frontier, KGW-NT reaches higher worst-case z-scores at the same PPL ratio and the same LLM-judge drop, or lower degradation at the same detectability. The gain over KGW-TK shows that sparsity alone is not the explanation: a top-*k* cutoff can still admit tokens with large logit gaps. On Llama-3.1-8B, KGW-TK and KGW-NT (q = 30, δ = 4) have similar output entropy (1.05 vs. 1.03) but PPL ratios of 1.20 vs. 1.02.

<details class="post-more" markdown="1">
<summary>All four models and both quality metrics</summary>

<figure class="post-figure post-figure--panels post-figure--four-panels">
  <div class="post-figure__scroll" tabindex="0" role="group" aria-label="Scrollable figure">
  <a href="/images/blog/near-tie-fingerprinting/frontier-all.png" aria-label="Open all-model results at full resolution"><img src="/images/blog/near-tie-fingerprinting/frontier-all.png" alt="PPL-ratio and judge-drop frontiers for all four models" loading="lazy"></a>
  </div>
  <figcaption>The trend holds under both metrics. On Gemma-2-9B, PPL differences are smaller because most baselines already incur little degradation there.</figcaption>
</figure>

</details>

### Result 2: query-efficient detection at low distortion

For each method and query budget N, we report the lowest PPL ratio among trained configurations whose worst-case z-score exceeds 4 within N queries. KGW-NT achieves the lowest PPL ratio at every evaluated budget. Larger budgets make sparser configurations detectable.

<figure class="post-figure post-figure--panels post-figure--three-panels post-figure--wide">
  <div class="post-figure__scroll" tabindex="0" role="group" aria-label="Scrollable figure">
  <a href="/images/blog/near-tie-fingerprinting/query-budget.png" aria-label="Open query-budget plots at full resolution"><img src="/images/blog/near-tie-fingerprinting/query-budget.png" alt="KGW-NT attains the lowest PPL ratio across evaluated query budgets on the three displayed models" loading="lazy"></a>
  </div>
  <figcaption><b>Figure 9.</b> Query-budget results on Llama-3.2-3B, Qwen-2.5-3B, and Llama-3.1-8B; Gemma-2-9B shows the same trend in the paper. On the 3B models, ratios below 1 indicate lower reference-model perplexity, with judge scores matching or exceeding the base.</figcaption>
</figure>

### Result 3: near-tie combines with existing watermarking schemes

Near-tie restricts the tokens receiving the bias. It can therefore be combined with SWEET’s selection of generation positions or MorphMark’s adaptive bias strength. Both combinations improve the detection–quality frontier in our evaluation.

<figure class="post-figure post-figure--panels post-figure--two-panels">
  <div class="post-figure__scroll" tabindex="0" role="group" aria-label="Scrollable figure">
  <a href="/images/blog/near-tie-fingerprinting/combine-ppl.png" aria-label="Open the combination plots at full resolution"><img src="/images/blog/near-tie-fingerprinting/combine-ppl.png" alt="SWEET and MorphMark frontiers with and without near-tie restriction" loading="lazy"></a>
  </div>
  <figcaption><b>Figure 10.</b> SWEET and MorphMark with near-tie restriction on Llama-3.1-8B. We apply q ∈ {10, 30, 50, 70} to one representative configuration of each scheme and compare with its 12 original configurations.</figcaption>
</figure>

## Conclusion

Prior utility evaluation understates what watermark-based fingerprinting costs in generation quality. Because verification aggregates evidence across queries, the watermark teacher need not be dense, and at matched strength the bias belongs on tokens the base model already finds plausible. Near-tie restriction applies this with a single logit-gap threshold. More broadly, watermark teachers for model fingerprinting should be designed for how fingerprints are verified, not inherited from text watermarking.
