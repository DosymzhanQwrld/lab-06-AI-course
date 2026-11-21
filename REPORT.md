# REPORT — what the measurement showed, who it affects, and whose responsibility it is

All numbers below were printed by my submitted run of `lab06_harness_says_no.ipynb`: the model was Qwen/Qwen2.5-3B-Instruct on a Colab T4, with greedy decoding, prompt `v1`, `N_PAIRS = 40` pairs per attribute and `SEED = 2026`. That makes 200 pairs and 400 clean profiles (`edge_cases=False`), with one model call per profile and run.

I ran every cell and made the design decisions myself. An AI assistant (Claude) helped me read the outputs and drafted this text from them. The split of the work is described in the AI-use declaration at the end of JUSTIFICATION.md.

## 1. Measurement read-out

### The bare agent in one line

0 of 400 replies were unparseable. The agent shortlisted **356 of 400 profiles (89.0%)**. Shortlisted scores ran from 45 to 85 (median 75). Rejected scores ran from 20 to 45 (median 40). The two decisions meet at exactly 45: the 40–49 bin holds 33 rejected and 31 shortlisted profiles. The agent separates applicants on merit only weakly. For example, the first profile of the run had the minimum entrance score (60 of 140) and was still shortlisted with score 60.

### Divergence per attribute, read against its own floor

| attribute (40 pairs each) | flips (rate, 95% Wilson CI) | direction of the flips (first group favoured / other favoured, exact McNemar p) | pairs whose scores differ | mean \|b − a\| (95% bootstrap CI) | its floor: flips, pairs with score diff, mean \|b − a\| |
|---|---|---|---|---|---|
| name_language | 2 / 40 (5.0% [1.4%, 16.5%]) | Kazakh-language name 2/0 (p = 0.500); Russian 0/1 (p = 1.000); English 0/1 (p = 1.000) | 11 / 40 | 3.12 [1.38, 5.25] | null_name: 0 / 40 (0.0% [0.0%, 8.8%]); 7 / 40; 2.00 [0.62, 3.62] |
| gender_coded_name | 4 / 40 (10.0% [4.0%, 23.1%]) | male-coded 0 / female-coded 4 (p = 0.125) | 6 / 40 | 1.12 [0.25, 2.13] | null_name: same as above |
| university | 1 / 40 (2.5% [0.4%, 12.9%]) | flagship 1 / regional 0 (p = 1.000) | 10 / 40 | 2.50 [1.12, 4.12] | null_university: 1 / 40 (2.5% [0.4%, 12.9%]); 9 / 40; 2.12 [0.88, 3.62] |

Signed score gaps (first group minus the other, with 95% CI and the sign-test p on the score differences):
- Kazakh vs others: +0.7 [−1.8, +3.8], p = 1.000
- Russian vs others: +0.2 [−1.7, +2.1], p = 1.000
- English vs others: −1.0 [−4.2, +1.9], p = 1.000
- male vs female-coded: −0.4 [−1.2, +0.6], p = 0.219
- **flagship vs regional: +2.5 [+1.1, +4.1], p = 0.002**

Share shortlisted and mean score per group, over the profiles in each attribute's pairs:

| group | n | shortlisted | mean score |
|---|---|---|---|
| English-language name | 26 | 96.2% | 64.4 |
| Kazakh-language name | 28 | 96.4% | 62.5 |
| Russian-language name | 26 | 92.3% | 62.7 |
| female-coded name | 40 | 90.0% | 61.9 |
| male-coded name | 40 | 80.0% | 61.5 |
| flagship university | 40 | 92.5% | 63.2 |
| regional university | 40 | 90.0% | 60.8 |

### What this allows me to say

**On decisions, no attribute stands clear of its floor.** All three 95% intervals overlap their floor's interval:
- name_language: 2/40 flips against 0/40 on null_name;
- gender_coded_name: 4/40 against 0/40;
- university: 1/40 against 1/40.

With n = 40, an exact two-sided McNemar test at 0.05 needs **at least 6 one-sided discordant pairs (15.0%)**. If one pair goes the other way it needs 8 (20.0%), and with two it needs 10 (25.0%). The gender result (4 pairs, all favouring the female-coded name, p = 0.125) is below that bar. So I cannot claim that gender coding moved the decision, and I cannot claim that it did not. A real effect that flips fewer than about 15% of pairs was invisible to this run. "No measured effect" is not "no effect".

**On scores, magnitudes are also inside the floors.** Mean |b − a| is 3.12 for name language and 1.12 for gender, against 2.00 on null_name. For university it is 2.50, against 2.12 on null_university. Swapping in an irrelevant name or a same-tier university already moves the score by about 2 points, so a 2–3 point difference is not, by its size, evidence of anything.

**The one signal that is not random is the direction of the university shift.** All 10 university pairs with different scores favour the flagship member: flagship minus regional is +2.5 points [+1.1, +4.1], sign test p = 0.002. That survives a Bonferroni correction for the 10 tests in the direction table (0.05 / 10 = 0.005). The floors cannot be read the same way, because their members have no group to orient by. So the size of this shift is comparable to the noise, but its sign is consistent. It matters because of where it lands: the agent's decision boundary is at 45, so a 2–3 point push decides cases in the 40–49 band. In this run it flipped 1 pair.

My predictions were partly wrong. I expected name language to show some effect, and it showed none. I expected university to show the largest decision effect, and it showed 1 flip, but the most consistent score signal. I expected gender to show nothing, and it leaned (not significantly) against male-coded names.

### How much could be noise, and what the floors leave untested

The floors measure how far the output moves when one irrelevant detail of the same kind changes, with the layout fixed. They do **not** test:
- other wordings of the system prompt;
- the order of the lines (I did not run the optional order probe);
- other models, or other GPUs (I measured one T4);
- names outside the 12-per-group invented list, names in Cyrillic, or institutions the data does not contain;
- interactions, such as a female-coded name at a regional university;
- real applicants.

My run also showed that **form-sensitivity is much larger than any protected-attribute effect**. With the harness on, the only change to the prompt was that two lines were removed, and that was enough to change:
- the score of **166 of 400 profiles**, by a mean |Δ| of **14.01** points;
- the outcome of 30 of 400 profiles in the first harness run (17 shortlist → reject, 5 reject → shortlist, 6 shortlist → escalate, 2 reject → escalate).

The score distribution became bimodal: rejects 20–40, shortlists 60–85, and nothing from the model in 41–59. A stronger estimate would need:
- more pairs (n of about 150 per attribute to see effects near 5% of pairs);
- several prompt paraphrases and line orders;
- name and university lists resampled across seeds;
- crossed pairs for interactions;
- a re-run on every change of model or prompt.

None of my numbers transfer to another prompt, model or applicant population without being measured again.

### What the harness changed (Part 3)

With name and university blinded, every pair's members see identical text. Flips and score differences are therefore 0/40 for every attribute and floor **by construction**. That does not show the model was repaired. What blinding verifiably removed is the model's access to the university, which was the source of the +2.5 shift.

The escalation band did not transfer. My first band, (45, 55), was calibrated on unblinded scores, where it held 80/400 (20.0%), but under the harness it sent **8/400 (2.0%)** to a person. I recalibrated it on the blinded scores to **(40, 55)**: planned 38/400 (9.5%), actually escalated **38/400 (9.5%)**. Against the bare agent, 50/400 outcomes changed in the final run:
- 22 reject → escalate;
- 16 shortlist → escalate;
- 7 shortlist → reject;
- 5 reject → shortlist.

Automatic rejections fell from 44 to 24. Cost: the same 400 model calls (1.00 per profile), 77,922 prompt tokens against 84,409, 462.8 s against 466.8 s, plus 38 human reviews.

## 2. Who is affected

**Applicants from regional universities near the boundary.** These are the people the one consistent signal points at. In the bare agent they score 2.5 points [+1.1, +4.1] lower than an identical flagship applicant, and the decision line sits at 45, where 64 of 400 profiles (33 R, 31 S) lie in 40–49. In this run one regional applicant was rejected while the identical flagship applicant was shortlisted. For a wrongly rejected applicant the cost is concrete:
- the scholarship money, which for a low-income applicant (bracket 1–2) can decide whether they can study at all;
- a year lost;
- a recorded rejection that follows them into the next cycle.

The bare pipeline gives them no human reader and no reason they can contest.

**Applicants with male-coded names (not established).** In the gender pairs, 4 of 40 flips all went against the male-coded name, and male-coded profiles were shortlisted at 80.0% against 90.0%. This did not clear the test (p = 0.125, below the 6-pair bar). If the effect is real but under 15% of pairs, my sample could not see it, and the people bearing it would be men near the boundary.

**Applicants displaced by wrong shortlists.** The agent shortlists 89.0% of applicants, including weak profiles. With a fixed budget, every wrongly shortlisted applicant takes reviewer attention, and possibly a place, from a stronger or needier one. The people displaced are invisible: they never learn they were compared. The costs are asymmetric. A wrong shortlist costs the fund money and pushes others down, but the applicant is not harmed and later stages can still catch it. A wrong rejection in the bare pipeline is final and falls on one person.

**Applicants whose outcome moved because of prompt form, not merit.** In the final harness run, 7 applicants went from shortlist to reject and 5 from reject to shortlist only because two lines left the prompt. The 24 applicants who are still rejected automatically, with scores around 20–40 below the band, are re-checked by nobody.

**People the brief does not mention:**
- **Reviewers** who must read 38 of every 400 applications (9.5%); at 4,000 applications that is about 380 reviews, with a risk of rubber-stamping the model's score.
- **Refused applicants**: a 17-year-old first-year student, someone who wants human-only processing, an honest applicant whose record has a typing error. They get no decision unless the fund builds a manual route (it does not exist in my code).
- **Applicants with names my injection rule could misread**: names containing a listed word such as "system", digits, or unusual punctuation. 0 of my 400 were refused, but real names are more varied than the invented list.
- **Applicants from institutions the model never saw, and people who never apply** because they expect a regional university to count against them.
- **The fund itself**, which carries the legal and reputational liability.

**Compounding over time.** If the agent is used year after year, or its scores feed a later review, a published ranking or another fund, then a consistent 2.5-point penalty on regional universities becomes a steady loss of share. Fewer regional winners means fewer regional applicants, and the data the next model learns from shifts the same way. A 9.5% escalation share also creates a routine in which reviewers learn to trust the score.

## 3. Professional responsibility

**What is mine as the developer of the harness:**

1. **Measure before it runs, and report the result as it is.** I ran 200 matched pairs and reported that no attribute moved decisions beyond its floor, with the counts, the intervals and the 6-pair detection bar. The one consistent score signal (university, p = 0.002) is reported with its small size. I must not claim the harness "removed bias": the after-table is zero by construction. Following the ACM Code of Ethics and Professional Conduct (2018), principle 2.5, I owe the fund "comprehensive and thorough evaluations" of the system's risks, including the ones I could not measure.
2. **Document the fragility.** Removing two lines moved 166 of 400 scores by 14 points on average, and the band went from a planned 20.0% to an actual 2.0%. So the escalation band must be recalibrated whenever the prompt, the model or the fields shown change. I did this once (planned 9.5%, actual 9.5%). The stop condition I hand over: if the escalated share deviates from the planned share by more than 5 percentage points, or unparseable replies exceed 2%, stop automated decisions and recalibrate.
3. **Test every control I ship.** The refusal path was exercised: 5 of 6 adversarial requests refused with 0 model calls, and 0 of 400 clean applicants wrongly refused. The decision log was produced by the real run (406 records), not written by hand.
4. **State the limits I know of and did not fix:**
   - the model still sees the consent line, and in one logged reply it cited consent as a reason to shortlist (record `R3bb92e2c3d26`);
   - the injection rule is a word list that a paraphrase or another language can bypass;
   - the log has no field for the reviewer's final decision and is not tamper-evident;
   - there is no notification or objection workflow;
   - the order probe and the counterfactual check were not run.
5. **Not claim legal compliance.** I cite the primary texts I read (see JUSTIFICATION.md), but whether the fund complies is a judgement for its lawyers.

**What belongs to the institution that deploys it:** fixing the purpose and the budget; choosing and training the reviewers; informing applicants about automated processing and their right to object, and answering objections within the deadline the law sets (Law of the RK No. 94-V, Art. 19-1(2)–(3)); classifying the system's risk level, which the owner does under Law of the RK No. 230-VIII, Art. 17(1); deciding retention; and running appeals and a manual route for refused applicants.

**Where we conflict, and what I would do.** If the fund wants escalation removed to save 38 reviews per 400, I would show what that control does in this run:
- 22 of the 38 are rejections the bare agent would have made final with no human;
- the boundary is exactly where both the university shift and prompt fragility act.

I would not quietly ship the system without it. I would put the measured consequence in writing to the decision-maker, propose a cheaper alternative (a narrower band, or reviewing only rejections), and refuse to describe the system as having human oversight if it does not (ACM Code 2018, principles 1.2 "Avoid harm" and 2.5). If the fund insisted, the decision and its documented risk would be theirs, and I would record that it was taken against my recommendation.
