# Kidney (NHANES), 472 of 500 cases

Qwen2.5-1.5B-Instruct, reasoning mode, both option orders, 1 sample per order, max 1024 new
tokens. Run on a Vast RTX PRO 4000 (about 27 s per case). The machine went offline at case 472,
so the last 28 cases are missing. 82 of the 472 patients have CKD. 81% of answers parse. Fit
with `scripts/extract_model_belief.py` (64/16/20 train/val/test split). Activations for all
472 cases are saved for the probe.

## How well each readout tracks the true CKD label (AUC, 0.5 = chance)

| readout | all cases | held-out test cases |
|---|---|---|
| stated probability | 0.61 | 0.63 |
| probability bin | 0.59 | 0.54 |
| Likert | 0.52 | 0.52 |
| z from reports only | 0.61 | 0.65 |
| z from decisions only | 0.65 | 0.67 |
| z from everything | 0.66 | 0.72 |
| raw fraction of "act" decisions | 0.62 | 0.62 |

The decisions carry a bit more information about the patient than the stated probability does,
and combining both is best. All signals are weak.

## Saying vs doing

- **Stated and decision-inferred beliefs barely agree.** Spearman correlation between
  `direct_prob` and `z_decisions` is 0.36.
- **Stated beliefs predict decisions only moderately.** Predicting each held-out decision from
  the reports alone gets 70% accuracy.
- **The numeric reports are very noisy.** The Beta precision φ sits at its lower bound (2.0).
- **Thresholds ignore the stakes.** For the four threshold questions the nominal cut-offs are
  33%, 20%, 10% and 5%, but the fitted ones are all about 8–10%. The model behaves as if it has a
  single screening threshold and does not read the number in the question.
- **Cost-ratio decisions** depend on belief (slope β 0.25–0.5) but act at fitted thresholds of
  about 0–2%, i.e. the model almost always says "screen".
- **Bets ignore belief entirely.** Slopes β are about 0 for all four bets, so the fitted
  thresholds (0 or 1) mean nothing.

## Position bias in the ordinal questions

Likert and the probability bin mostly pick the last option in both orders, so they flip
meaning when the options are reversed. The reasoning often contradicts the letter it picks, e.g.
it concludes "an increased likelihood of having chronic kidney disease" and outputs
"Choice: E. Very unlikely". The answer mapping is correct (checked `choice_shown` against
`answer`). Averaged over the two orders, these two readouts carry almost no signal (AUC 0.52
and 0.59), and the bin correlates only 0.09 with the stated probability.

## Next

- Code (`insecure_code_em`) and werewolf (`werewolf_games`) runs, on a new machine.
- Linear probe (readout 3) on the saved activations, trained to match `z_all`.
- Read the ordinal answers from `option_probs` instead of the text, averaged over orders.
