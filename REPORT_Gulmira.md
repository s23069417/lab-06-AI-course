# REPORT — what the measurement showed, who it affects, and whose responsibility it is

Replace this whole template with your own text. Delete the line above (the `STUDENT: replace`
marker) once you have, and delete these instructions and the guiding questions too. Keep the three
numbered headings. Every number you quote must be one **your own notebook printed**, copied exactly,
with the group it refers to (which attribute, how many pairs). Which dimension each section is
graded against is in the README.

The guiding questions are prompts, not a form: you do not have to answer each one in turn, and
answering them without evidence from your run earns nothing.

## 1. Measurement read-out

(Dimension: Evidence of Specific Impact.)

Guiding questions:

I ran 40 matched pairs for each of five groups (name language, gender-coded name, university, and the two floors null_name and null_university): 200 pairs, 400 profiles, 400 model calls, with Qwen2.5-3B-Instruct under greedy decoding. 356 of 400 profiles were shortlisted. 0 of 400 replies were unparseable.

**Flips and direction (bare agent).**

| attribute | flips | rate [95% CI] | its floor | direction |
|---|---|---|---|---|
| name_language | 2/40 | 5.0% [1.4%, 16.5%] | null_name 0/40 (0.0% [0.0%, 8.8%]) | Kazakh-name member favoured 2, other 0 (McNemar p=0.500) |
| gender_coded_name | 4/40 | 10.0% [4.0%, 23.1%] | null_name 0/40 | male-coded favoured 0, female-coded favoured 4 (p=0.125) |
| university | 1/40 | 2.5% [0.4%, 12.9%] | null_university 1/40 (2.5% [0.4%, 12.9%]) | flagship favoured 1, regional 0 (p=1.000) |

Every flip was a shortlist-versus-not flip. There were 0 "other flips".

**Score shifts.** The mean absolute score difference |b − a| against its own floor was:

- name_language: 3.12 [1.38, 5.25] in 11/40 pairs, against 2.00 [0.62, 3.62] in 7/40 for null_name.
- gender_coded_name: 1.12 [0.25, 2.13] in 6/40 pairs, against the same null_name floor.
- university: 2.50 [1.12, 4.12] in 10/40 pairs, against 2.12 [0.88, 3.62] in 9/40 for null_university.

All three intervals overlap their floors. The signed score gaps, first group minus the other, were:

- Kazakh vs others: +0.7 [−1.8, +3.8]
- Russian vs others: +0.2 [−1.7, +2.1]
- English vs others: −1.0 [−4.2, +1.9]
- male-coded vs female-coded: −0.4 [−1.2, +0.6]
- flagship vs regional: +2.5 [+1.1, +4.1], sign p=0.002

**What this supports.** No attribute stands clear of its own floor on flips or on absolute score difference. Gender is the closest: 4/40 against a floor of 0/40, all four against the male-coded name (shortlisted 80.0% male vs 90.0% female). But the intervals overlap and p=0.125, so I do not call it an effect. The university score gap is the only signed result whose interval excludes zero. It is small (+2.5 points), it moved one decision, and I did not print a signed gap for null_university. I therefore cannot say whether a same-tier swap would produce a similar shift. Several rows of tests were read, so a p-value of 0.002 in one row should be read with care.

**What the floors test and do not test.** null_name changes the name within the same language group and gender coding. null_university changes the university within the same tier. Both keep fixed the prompt wording, the field order, the model, the merit values and the invented name and university lists. A floor of 0/40 is not zero: its upper bound is 8.8%. A stronger estimate would need more pairs, several draws of name pairs, several prompt wordings and layouts, other models, and real applications.

**What 40 pairs could detect.** The exact McNemar test needs at least 6 pairs (15.0% of 40) with none the other way, 8 with one the other way, and 10 with two the other way. My largest count was 4. An effect that moves fewer pairs than that could be real and invisible here. "No measured effect" is therefore not "no effect". The room for flips is also small: with 356 of 400 shortlisted, most profiles are already on one side of the decision.

**The largest instability I found was not an attribute.** In the optional order probe, 40 pairs of identical applicants, one shown with its lines in another order, gave 10 flips = 25.0% [14.2%, 40.2%]. All 10 went the same way (10 vs 0, p=0.002), and the mean score was −6.6 [−9.9, −3.5] in the other order. Layout alone moved more decisions than any attribute in Part 1. The floors in Part 1 do not capture this, because both members of a floor pair are laid out alike.

**What generalises.** My run cannot support claims about a different prompt wording, another model, real applicants or real names. The data are invented, one model was used, there was one prompt version (v1), and the order probe shows the result depends on layout.


## 2. Who is affected

(Dimension: Affected Parties and the Consequences They Bear.)

**Wrongly rejected.** The only directional pattern I measured was against male-coded names (4 flips, 0 the other way; 80.0% shortlisted vs 90.0% for female-coded) and in favour of flagship universities (+2.5 score points; 92.5% vs 90.0% shortlisted). Neither flip count is above its floor, so I cannot say these groups are harmed. I can say that if there is an undetected effect of the size my sample could miss, it would fall on male-coded applicants and on applicants from regional universities. Separately, and regardless of attribute, anyone whose profile is formatted differently is exposed to the 25.0% order-probe divergence. A person wrongly rejected loses a place and possibly funding. The model returns a one-sentence reason, but my run does not show whether the applicant ever sees it.

**Wrongly shortlisted.** With 356 of 400 shortlisted, the shortlist barely filters. I do not know how many places the fund has, so I cannot say who is displaced. What I can say is that any wrongly shortlisted applicant takes a place from someone ranked below them. That cost falls on a person I cannot see in my data.

**Costs are not symmetric.** A wrongly rejected applicant loses a place they were due. A wrongly shortlisted one gains it (a later withdrawal or reputational cost is an assumption; I did not measure it). The first cost is harder to detect, because the rejected person rarely sees the profile that displaced them.

**Applicants the harness itself changed.** Comparing the harness run with the bare run, 46 of 400 decisions changed: 22 reject → escalate, 12 shortlist → escalate, 7 shortlist → reject, 5 reject → shortlist. The 34 escalations come from the escalation control. The 12 changes between shortlist and reject (in both directions) happened when name and university were removed from the prompt; I cannot tell whether they reflect reliance on those fields or only the changed form of the prompt. 166 of 400 scores changed, by a mean of 14.01 points. Either way, 12 applicants got a different shortlist/reject outcome from two configurations of the same system, and 22 rejected profiles now get a human reader they would not have had.

**Parties the brief does not mention.**

- Reviewers: 34 of 400 profiles (8.5%) went to a person. They carry the workload, and their reading becomes the real decision.
- Applicants refused by my rules, for example a 17-year-old (reason minor), who gets no decision at all. In Part 4 my harness refused 5 of 6 requests, and none of the 400 clean applicants was refused. But all 400 names came from the lab's own list, so this does not show the name rule is safe on real names.
- People who never apply because they expect to be rejected, and applicants from institutions or name groups the model has never seen. My data cannot show either.

**What compounds over time.** If the agent is used every year, the same sensitivity to layout repeats across cohorts. If its output feeds another stage (a later review, another fund, a ranking), a 25.0% layout-driven divergence becomes an input rather than a one-off error. Reviewers may also be anchored by the model's score and reason if they see them; my run does not test this.


## 3. Professional responsibility

(Dimension: Recognition of Professional Responsibility.)
**What I must check before it runs.**

- Test sensitivity to layout, not only to protected attributes. My floors missed what the order probe found (25.0%).
- Recalibrate the escalation band on the harness's own scores. I planned 16.0% (64 of 400) and got 8.5% (34 of 400), because the band (40, 49) was calibrated on unblinded scores.
- Refuse requests the system must not serve. In Part 4 the bare pipeline screened all six requests, including purpose "rank by ethnicity" (shortlist) and a name field containing an instruction (shortlist).

**What I must tell the people who deploy it.**

- The result is "no measured effect beyond the floors with 40 pairs", not "no bias". 40 pairs cannot detect fewer than 6 one-sided pairs (15.0%).
- Blinding makes flips zero by construction (0/40 in all five groups after the harness). That table is not evidence that the harness removed a bias.
- The harness changed 46 of 400 decisions relative to the bare run (34 became escalations through the escalation control; 12 moved between shortlist and reject after name and university were removed) and 166 of 400 scores (mean 14.01 points). So it is not neutral, and the band must be recalibrated after any prompt change.
- Escalation decides who looks at a case. It does not make the decision fair.

**What I must not claim.** I must not claim the system is fair, that blinding fixed a measured bias, or that the model "does not use" name or university. I measured none of these. I must not present the counterfactual extension as validation: it ran on 8 pairs (16 profiles, 32 model calls) and gave 14 shortlist and 2 escalate, both escalations for the borderline band and none for counterfactual disagreement.

**What discharging these would require in practice, and what is done and left.**

| responsibility | done in this lab | left |
|---|---|---|
| floor-based measurement | yes (40 pairs per group) | more pairs, more floors |
| layout test | order probe run (25.0%) | fix or mitigate layout dependence |
| refusal path | exercised (5 of 6 refused, 0 model calls) | test on real names |
| log | 406 records in decisions.jsonl | retention and access rules |
| escalation | band set, share measured (8.5%) | recalibrate, define reviewer procedure |
| appeals | not built | institution to define |

**What belongs to the institution.** The institution sets the purpose of the system, how many places there are, who reviews escalated cases and how they are trained, the appeals route for rejected and refused applicants, and the retention rules for the log. I cannot set these from inside the harness.
**Where the two conflict.** Suppose the institution asks me to remove refusals or escalation to save reviewer time or cost. I measured what the refusal path prevents: without it, the bare pipeline screened all six requests, including one that serves a purpose the system must not serve. Escalation sends 8.5% of applicants to a person at no extra model cost (400 calls in both runs; 485.2 s harness vs 494.9 s bare), and its price is reviewer workload. The counterfactual check would double the model calls (32 calls for 16 profiles in my extension). It caused no escalation there (both escalations were borderline), but 8 pairs are too few to say whether it would catch anything, so I cannot use that run to argue the control is unnecessary. My responsibility is to document what each control prevents, to say what my evidence does not show, and to not remove a control silently or to describe the system as validated once it is removed. If the institution still removes it, that decision and its owner must be written down.
