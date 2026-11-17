# Retro: Virtual Cell Challenge 2026

I did the [Virtual Cell Challenge 2026](https://virtualcellchallenge.org/): CRISPRi knockdown effects in an unseen cell type. Arc Institute knocks the genes down and measures every gene in single cells, and you have to say what a few hundred knockdowns do in a cell type you have not seen. The final test is three new cell lines, and all you get is their control cells.

Our best was 0.164, about 463rd of 1,580 when validation closed on October 22. The top ten were near 0.32. Illumina was at 0.47. That one, billmA, is what we submitted at the end. The final numbers are not out yet.

## What we did

We did not measure anything ourselves. We use what is already measured in the public screens, and we move it to the control cells of the new cell type.

The first one, v1, was 0.0877: we added the public signatures for the same target to the control mean. Then I did the same average shift on every target, with the signatures shrunk, and that one was much worse, -0.2482. It was a bad idea. v4a was 0.1438, and that was when we started from [AtlasShift](https://github.com/kaipengm2/Virtual-Cell-Challenge-2026), a public method. v5b was 0.1528, when we matched the spread to how the scorer credits MSE, and only kept the gene calls that the screens agree on. v9a was 0.1613, with the gene calls in the order of how confident we are. billmA was 0.1644, and that is our version of a public response-network idea.

Most of the gain was from v1 to v4a. That was when I stopped building on my own first model and started from a public one. After that it was small steps. Every entry that was scored is in [OFFICIAL_SCORES.md](OFFICIAL_SCORES.md).

## The baselines and the metric

There are six things in the score. Most of what moved ours was discrimination: can you tell which knockdown it is, out of a lineup. It was 0.52 in v1, 0.68 with AtlasShift and 0.71 with billmA. The MSE part of it was 0 all the way, until we matched the spread of what we predict to how the scorer credits it.

Discrimination is the easy part of this metric. Direction is the hard part. On 168 training targets our discrimination was 0.86, and the mean cosine with the real response was 0.084. So it can be good enough to pick the right knockdown and still barely point the right way.

The simple ones are hard to beat here. [Ahlmann-Eltze, Huber and Anders](https://doi.org/10.1038/s41592-025-02772-6) showed it: on perturbation prediction, the deep models do not beat the simple linear ones yet. Arc said the same thing in their own [2025 wrap-up](https://arcinstitute.org/news/virtual-cell-challenge-2025-wrap-up). And the best step we did was a public baseline, not one of my models.

## What we learned about the problem

Two halves of the same real cells agree at about 0.65, so the 0.084 is not noise. We were at about a tenth of what the data allows ([noise ceiling](bio_gap_1026/ceiling/REPORT.md)).

It was not that some targets had no data. The targets that did have public data carried 99.5% of the error. The public data is from other cell types, and that is what was missing: data from the same cell type.

If you have other knockdowns in the same cell type, alignment goes from 0.115 to 0.42. The final phase takes exactly that away. So we did the same thing to ourselves and held out whole cell types, and we were at 0.118, the same as before ([biology diagnostics](bio_gap_1026/REPORT.md)).

Most of the fixes that made the error smaller did it by moving what we predict to 0, which the MSE part of the score does not pay you for.

So with public data from other cell types, I think a new cell line is mostly a data problem, not a modelling problem. The leader has a private [atlas of more than 200 cell lines](https://www.illumina.com/company/news-center/press-releases/2026/fda84c92-b4b3-4691-a402-35555abe8605.html). I could be wrong. If one of the top final entries used no data from the same cell type and still did better than 0.3, then it is a modelling problem and I had the bottleneck wrong.

## What was easy and what was hard

Easy: to get started. The screens are public, the good methods are public, and so is the scorer, [cell-eval2](https://github.com/ArcInstitute/cell-eval2).

Hard: to know if a change helped, after validation closed. We ran the official scorer here on held-out training targets and we thought it would be close enough. It was not. It said one change was a clear win (+0.066) that had already been scored as a small loss (-0.009). Of 4 changes that were scored both ways, it got the direction right on 2 ([calibration](calib3_1026/REPORT.md)). After that we tried to calibrate the size and the direction, more gene calls and fewer gene calls, and a small test of network features. None of it held up.

Our biggest gap was expression error, about 0.06 against about 0.44 for the entries ranked 10 to 50. By the time we got to it we had no scorer we could trust to tell us if it helped.

## What we did not have

- The same knockdowns measured in a lot more cell lines.
- The controls and the repetitions next to them, so that everyone can see the noise ceiling per target.
- How much each target really went down.
- A public held-out set that ranks the entries the way the official one does.

## What I would do differently

Start from the best public baseline. Then build the local scorer first, and check that it ranks the entries we already have the way the official one did, while it can still answer you. Ours would not have passed that. Keep one cell type that you do not touch until the end. And I would treat the close of validation as the end, because after that we could not test anything.

## Still open

Was the leaders' edge data from the same cell type? I do not know. And how do you move the size of a response into a cell type where all you have is the controls? And which of the public screens add signal of their own?

## Thanks

- Arc Institute, for the challenge and for the scorer, [cell-eval2](https://github.com/ArcInstitute/cell-eval2).
- kaipengm2, for [AtlasShift](https://github.com/kaipengm2/Virtual-Cell-Challenge-2026). It was our first real baseline.
- Saber HQ, for [Sidechain](https://github.com/saberhq/sidechain).
- Bill M. His response network became our best entry.
- Matin Gerami, for [zero-shot-perturbation](https://github.com/MMatinGerami/zero-shot-perturbation), [danielchen26](https://github.com/danielchen26/vcc2026-stage1-lab), and the Rasmussen Lab for [SLIM](https://github.com/RasmussenLab/SLIM).

The code and the reports are in this repo. There is no data and there are no model weights.

## References

- Everything we did and what it was worth, in this repo: [OFFICIAL_SCORES.md](OFFICIAL_SCORES.md), [bio_gap_1026/REPORT.md](bio_gap_1026/REPORT.md), [bio_gap_1026/ceiling/REPORT.md](bio_gap_1026/ceiling/REPORT.md) and [calib3_1026/REPORT.md](calib3_1026/REPORT.md).
- Ahlmann-Eltze, Huber and Anders, Nature Methods (2026), on the deep models and the simple ones: [doi:10.1038/s41592-025-02772-6](https://doi.org/10.1038/s41592-025-02772-6)
- Arc Institute, Virtual Cell Challenge 2025 wrap-up: [arcinstitute.org/news/virtual-cell-challenge-2025-wrap-up](https://arcinstitute.org/news/virtual-cell-challenge-2025-wrap-up)
- AtlasShift and the scorer, cell-eval2: [github.com/kaipengm2/Virtual-Cell-Challenge-2026](https://github.com/kaipengm2/Virtual-Cell-Challenge-2026), [github.com/ArcInstitute/cell-eval2](https://github.com/ArcInstitute/cell-eval2)
