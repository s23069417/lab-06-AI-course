# JUSTIFICATION — the design of the harness, and why

Replace this whole template with your own text. Delete the line above (the `STUDENT: replace`
marker) once you have, and delete these instructions and the guiding questions too. Keep the headings
that follow. Which dimension each section is graded against is in the README.

Two rules for the whole document:

- **Every claim about a law needs a citation to the instrument and the provision** it rests on,
  written so a reader can find the text. You identify the instruments yourself, from the primary text.
  A chat model's statement is not a citation.
- **Everything you describe must be in the code or in the submitted run.** A control you describe
  but did not build, or a refusal you state but never triggered, belongs under "what the design still
  fails to protect against".

The guiding questions are prompts, not a form.

## 1. What the system may do

(Dimension: Integration of Constraints into the Design.)

**Purpose.** The system screens applications for the Tulpar Scholarship Fund. It reads one applicant profile and answers *shortlist* or *reject* with a score from 0 to 100 (this is the system prompt of the supplied agent). It does nothing else. In my harness run of 400 profiles the model alone decided 366 of them (342 shortlist, 24 reject). 34 profiles (8.5%) went to a person, and none were refused, because the measured applicants are clean. A person looks only at the escalated cases. Nobody re-checks the other 366 (see section 6).

**The five controls, the constraint behind each, and where they are in the code.**

| control | constraint that made me build it | where in the code |
|---|---|---|
| `validate_input` | the system must not decide on impossible data (AI Law, Art. 11(1) reliability) | `Harness(validate_input=True)`, `validate_profile`; fired on "GPA 9.9, exam score -5" |
| `blind` | no discrimination by language or sex (AI Law, Art. 6); Part 1 found a +2.5 point university score gap | `BLIND = ("name", "university")` in 2a, `Harness(blind=BLIND)`; the model is shown 7 fields (`fields_shown` in the log) |
| `refuse` | no automated processing without consent (PD Law, Art. 19-1(1)); minors; wrong purpose; instructions in free text | `Harness(refuse=True)` (purpose, minor, consent are built in), plus my own rule `my_extra_refusal` in 2b |
| `log` | someone must be able to see why a decision was made (AI Law, Art. 16(1)(5), Art. 15(2)(3)) | `log_path="decisions.jsonl"` |
| `escalate` | a person must be able to check uncertain calls (AI Law, Art. 8(1), Art. 17(2)) | `escalate=True`, `borderline=(40, 49)`, plus my own rule `my_extra_escalation` in 2c |

**Constraints that changed the design.**
- The refusal rules run before the model is called. In Part 4 the five refused requests used 0 model calls. Without this, the bare pipeline screened all six requests, including the purpose "rank applicants by ethnicity" (it answered shortlist) and the name field that contains an instruction (also shortlist).
- I chose the escalation band (40, 49) from the score histogram of Part 1 and not from a round number. That bucket held 33 rejects and 31 shortlists, so it is exactly where the model could not decide. Without the oversight constraint I would have left all decisions to the model.


## 2. What it must refuse

(Dimension: Refusal Boundaries and Enforcement.)

| request | reason code | enforced by | fired in my run |
|---|---|---|---|
| purpose is not scholarship screening (`rank_by_ethnicity`) | `purpose_not_allowed` | built-in rule of `screen` | Part 4, and log record `R946c58f34eb5` |
| applicant is 17 | `minor` | built-in rule | Part 4 |
| no consent to automated processing | `no_consent` | built-in rule | Part 4 |
| impossible values (GPA 9.9, exam score -5) | `invalid_input` | `validate_input` | Part 4 |
| the name field is an instruction | `instruction_in_free_text` | my own rule `my_extra_refusal` | Part 4 |

The harness refused 5 of the 6 requests (the sixth was an ordinary application, which went to the model). The bare pipeline refused 0 of 6 because it has no refusal path. The log has 5 refusal records.

**Which legitimate case does each refusal also block, and who pays?**
- *Purpose:* I cannot name a legitimate case. The system is only meant for scholarship screening.
- *Minor:* a 17-year-old with a legitimate application gets no decision at all. The law would allow consent through a legal representative (PD Law, Art. 8(1)), but my prototype has no way to collect or check it. The cost is paid by young applicants. I accept it for a prototype, but it is a real cost and the next step is to send these cases to a person.
- *No consent:* an applicant who did not tick the box gets no decision. The law requires consent for this kind of processing (PD Law, Art. 19-1(1)), so I accept the cost.
- *Invalid input:* an honest applicant with a typing mistake in the GPA is refused although the mistake is not theirs.
- *Instruction in free text:* my rule refuses a name with a dot or a digit, or a name or university that contains a word from my list (`system`, `score`, `output`, `prompt`, `reject`...). Of the 400 clean applicants in the measured pairs my rules refused 0. This is a weak test, because all 400 names come from the lab's own list. A real applicant with an unusual name could be refused wrongly and would pay the cost.

**What happens to a refused request.** It is recorded in `decisions.jsonl` with a reason code, with `raw_output` set to "[no model call]", empty `fields_shown`, and no score. So a reviewer later can see that no decision came from the model. But the applicant is not told, and there is no way to ask for a person to decide instead. I did not build that (see section 6).


## 3. What record it keeps

(Dimension: Accountability Record and Auditability.)

**Audit question:** "Why did this application get this outcome, and could someone else reconstruct it without re-running the model?"

**A record answers it.** Take log record `Re4c2a22e81f8` (application `P0043a`). It shows that the model saw 7 fields (`program`, `age`, `gpa`, `exam_score`, `household_income_bracket`, `achievements`, `consent_to_automated_processing`) and that `name` and `university` were withheld. It shows the model replied `reject` with score 40 and the reason "Low GPA and entrance exam score for Mechanical Engineering". Because 40 is inside the band (40, 49), the outcome is `escalate`, reason code `borderline`, controls fired `blind`, `escalate`, `log`, and `escalated_to_human` is true. A reviewer can see from the record alone that the model leaned to reject, why it went to a person, and which settings were on (the harness settings are stored in the record). The refusal record `R946c58f34eb5` shows `[no model call] purpose='rank_by_ethnicity'`, outcome `refuse`, reason `purpose_not_allowed`.

**What the log holds.** 406 records: 400 for the measured run and 6 for Part 4 (outcomes: shortlist 343, reject 24, escalate 34, refuse 5). Each record has a record id, the application id, a SHA-256 hash of the exact text the model saw, the names of the fields shown and withheld, the model id (`Qwen/Qwen2.5-3B-Instruct`), the prompt version (`v1`), the harness settings, the raw reply, decision and score, the controls that fired, the outcome and reason code, whether it went to a person, and a timestamp.

**What I left out on purpose.** The values of the fields (name, GPA, and so on). The log has field names only, so it does not become a second copy of the applicants' data. The cost is that the hash proves what the model saw only if someone also has the profile, and the log alone cannot rebuild the prompt text. One more limit: the raw reply can quote what the model was shown (in the record above it names the program), so the log is not completely free of applicant data.

**What I did not build.** I set no retention period, no rules about who may read the log, and nothing that stops an edit after the fact. `decisions.jsonl` is a plain file. I deleted and recreated it at the start of Part 3 (`Path(LOG_PATH).unlink(...)`), which shows how easy it is to change. The log also does not record what the human reviewer decided about the 34 escalated cases, so the chain stops at "a person was asked".


## 4. The legal constraints

**Before the search:**
Prediction: I think validate_input will be the hardest control to tie to a specific provision. It is a sanity check: is the GPA a number between 0 and 4, is the age an integer, is consent a real boolean. This looks like engineering hygiene rather than a regulated activity. I expect a data-protection law to say at most that data must be accurate or kept secure, and I do not expect any article to say "check that GPA is a float". So I expect to find only a general clause on accuracy or integrity of processing. That said, validate_input does more than check types: it refuses to make a decision on impossible values, so it keeps the system from deciding on garbage data. If the clause I find does not bind this prototype, I will mark it in the register as a benchmark.

(Dimension: Identification and Citation of Legal Constraints.)

**After the search, the prediction was mostly right.** I read the primary text of both laws. No article requires input validation. I expected to find an accuracy principle in the personal data law, but its Art. 5 lists only legality, constitutional rights, confidentiality, equality of rights and security. The closest text is in the AI law: Art. 11(1) (systems must be safe and reliable, excluding unforeseen consequences) and Art. 15(2)(2) (the owner protects the system from failures). So `validate_input` is tied to a general reliability clause, and I mark it as a benchmark.

**Which instruments, and why they apply.** I assumed the fund operates in Kazakhstan (the lab is run in Almaty), so I used Kazakh law. The fund and all applicants are invented, so for this prototype the laws are a benchmark and would bind a real deployment.
- Law of the Republic of Kazakhstan No. 230-VIII of 17 November 2025 "On Artificial Intelligence" (in force since 18 January 2026). It applies because the system is an AI system that scores people.
- Law No. 94-V of 21 May 2013 "On Personal Data and Their Protection". It applies because applicant profiles are personal data.

I read the AI law as amended on 25 August 2026 (Law No. 326-VIII of 24 June 2026) on the legal database Paragraph (prg.kz), and the personal data law on a legal database, both on 21 November 2026. I compared the amended AI law with its original text of 17 November 2025. The amendment changed only terminology (Art. 1 point 5, Art. 13(1)(6), Art. 17(1), Art. 20(1) and Art. 28(1); for example "information object" became "digital object"). The articles I cite (Art. 4, 6, 7, 8, 9, 11, 15, 16, 17(2), 18 and 21) have the same wording in both versions.

**Where the texts speak about automated decisions.** The personal data law defines automated processing as processing without the participation of the owner or operator (Art. 1, point 2-3). Art. 19-1(1) forbids automated processing that creates, changes or ends a person's rights unless the person consented, and Art. 19-1(2)-(3) require the operator to explain the procedure and allow an objection, which must be answered within three working days. The AI law points to this itself: Art. 21(4) says the requirements for decisions based only on automated processing of personal data are set by the personal data law, and Art. 7(2) gives the person the right to be informed.

**What the law requires, what it only permits.**
- *Requires:* consent for automated processing (PD Law, Art. 19-1(1), also Art. 7(1) and Art. 8(1)) → `refuse`; non-discrimination (AI Law, Art. 6) → `blind`; continuous control by the owner (Art. 8(1)) → `escalate`; documentation of the system (Art. 15(2)(3)) and the user's right to ask for the data behind a decision (Art. 16(1)(5)) → `log`.
- *Permits:* the AI law lets a medium-autonomy system decide on its own as long as a human can correct or override it (Art. 17(2)). How wide a band to send to a person is left to me.
- *Not built although the law asks for it:* the explanation of the procedure to the applicant and the channel for an objection (PD Law, Art. 19-1(2)-(3); AI Law, Art. 7(2)). My system does not tell applicants anything. I list this as a gap in section 6.
- *Where I did more than the law:* the refusal of every applicant under 18. Art. 8(1) would let a legal representative give consent. I also hide fields from the model, which the law does not ask (it forbids discrimination but does not say how to avoid it).


## 5. The ethical principles

(Dimension: Identification of Applicable Ethical Principles.)

I took the principles from two sources. First, Art. 4 of the AI law lists seven principles of state regulation (legality; fairness and equality; transparency and explainability; responsibility and controllability; priority of human well-being and free will; data protection and confidentiality; safety and security). This is a binding statute, but it states principles for regulation, not for my fund. Second, the UNESCO Recommendation on the Ethics of Artificial Intelligence (2021), section III.2 "Principles". It is a recommendation of the UNESCO General Conference: it is not a treaty and creates no binding obligation.

| principle | the decision in this system it bears on | source naming it |
|---|---|---|
| fairness and non-discrimination | hiding `name` and `university` (`blind`); reading each attribute against its floor in Part 1 | AI Law, Art. 4(2) and Art. 6; UNESCO III.2 |
| human oversight and determination | sending scores in (40, 49) and unparseable replies to a person (`escalate`) | AI Law, Art. 4(4)-(5), Art. 8(1); UNESCO III.2 |
| transparency and explainability | one log record per call with reason code (`log`) | AI Law, Art. 4(3), Art. 16(1)(5); UNESCO III.2 |
| right to privacy and data protection | refusing without consent (`refuse`); the log stores field names, not values | AI Law, Art. 4(6); UNESCO III.2 |
| safety and security | refusing impossible values and instructions in the name (`validate_input`, my refusal rule) | AI Law, Art. 4(7), Art. 11; UNESCO III.2 |
| protection of the vulnerable | refusing applicants under 18 | AI Law, Art. 9 (human well-being and free will); UNESCO III.2 |

**Where the principles conflict.**

1. *Treating applicants identically versus treating them fairly given what I measured.* Blinding makes the two members of every pair identical, so the flips after the harness are 0/40 in all five groups. This is zero by construction and does not show that the harness worked on the model. In Part 1 no attribute stood clear of its floor: name_language 2/40 (5.0%, CI 1.4-16.5%) and gender_coded_name 4/40 (10.0%, CI 4.0-23.1%) against a floor null_name of 0/40 (0.0%, CI 0.0-8.8%); university 1/40 (2.5%) against null_university 1/40. The only clear signal was a score gap of +2.5 points for flagship over regional universities (CI +1.1 to +4.1, sign p=0.002), and all 4 gender flips went against the male-coded name (McNemar p=0.125, not significant). Blinding cost something: 166 of 400 scores changed (mean change 14.01 points), and 12 decisions flipped between shortlist and reject (7 and 5, in both directions). I put non-discrimination first, as a precaution, because the cost fell on decisions in both directions and not on one group. But I cannot claim blinding made decisions fairer: the fields `program`, `age` and `household_income_bracket` are still shown, and Part 1 did not test them as proxies.
2. *Human oversight versus reviewer workload.* A wider band means more cases seen by a person but more work. I planned 16.0% (64 of 400) and got 8.5% (34 of 400). I put oversight first for the borderline scores, but I accepted that cases outside the band are not checked.
3. *Protection of minors versus access.* Refusing under-18s protects them, but also blocks a 17-year-old with a legitimate application. I put protection first because the prototype cannot check a guardian's consent.

**Where ethics demands something the law does not.** The refusal of every applicant under 18. PD Law Art. 8(1) allows consent by a legal representative, so the law would not force me to refuse. I refuse because I have no way to check the representative, and a wrong decision about a minor seems worse to me than no decision. I did not count "testing matched pairs before deployment" as an example: AI Law Art. 18(1) asks the owner to identify and assess risks, which may already cover it.


## 6. The trade-offs

(Dimension: Trade-off Analysis and Justification.)

**Cost of each control, using my before-and-after numbers.**

| control | cost | harm it prevents |
|---|---|---|
| `blind` | 166 of 400 scores changed (mean 14.01); 46 of 400 outcomes changed (reject → escalate 22, shortlist → escalate 12, shortlist → reject 7, reject → shortlist 5); 0 extra model calls (400 vs 400); time 485.2 s vs 494.9 s for the bare run | the model reading name and university; the +2.5 point university gap is gone because the model no longer sees the university |
| `validate_input` | 1 refusal in Part 4, 0 of 400 clean applicants refused | a decision on impossible values (the bare pipeline returned reject for GPA 9.9) |
| `refuse` | applicants who are refused get no decision; 0 of 400 clean applicants refused, but a weak test | five requests that should not be screened; they used 0 model calls |
| `log` | 406 records; stores field names only; I did not measure its cost separately | decisions that nobody can reconstruct |
| `escalate` | 34 of 400 profiles (8.5%) for a reviewer to read | uncertain calls decided by the model alone (the 40-49 bucket had 33 rejects and 31 shortlists) |

**The escalation band.** I planned that (40, 49) would send 64 of 400 baseline profiles (16.0%) to a person. In the harness run 34 of 400 (8.5%) went. It is lower because I chose the band on the scores of the unblinded model, and the scores moved by 14 points on average when the prompt got shorter. I did not recalibrate the band after blinding: it is still (40, 49). So what the band means has changed, and I should recalibrate it on the harness scores. The code does not say who reads the 34 cases or how fast. Outside the band, 366 decisions (342 shortlist, 24 reject) are never re-checked. My own escalation rule (reason mentions a protected attribute, decision does not fit the score, strong applicant rejected) could catch some of them, but in my run it fired 0 times: all 34 escalations have the reason code `borderline`.

**Why this balance, and what would change it.** I kept the harness cheap: no extra model calls, and the controls are ordinary code. If reviewers could read more, I would widen the band. If a larger run (more than 40 pairs per attribute) put an attribute clearly above its floor, I would hold name and university back and look at the proxy fields too. With n=40 I could only see an effect if at least 6 pairs flipped in one direction and none in the other, and my largest count was 4, so a small effect could have been missed. Also the agent shortlisted 356 of 400 (89%), so there was little room for flips.

**What the design still fails to protect against.**
- Proxy fields (`program`, `age`, `household_income_bracket`) are still shown and were never tested.
- Part 1 is small and synthetic: 40 pairs per attribute, one model, greedy decoding, one prompt version. "No effect found" is not proof of no bias.
- Applicants are not informed and have no objection channel, and refused applicants have no way to ask for a person (PD Law Art. 19-1(2)-(3); AI Law Art. 7(2)). Closing this needs a notice and a reviewer queue, which I did not build.
- The log has no retention period, no access control and nothing that stops edits, and it does not record the human reviewer's decision.
- My refusal rule is a word list on `name` and `university` only. It can refuse honest applicants, and it does not check other free-text fields such as `program`.
- 366 decisions outside the band are not reviewed by anyone.
- I did not run the counterfactual check or the order-sensitivity probe, so I do not know how much of what I measured is sensitivity to prompt format.

I did not close these gaps because the lab is a prototype on invented data and the time was limited, but each of them would have to be closed before a real fund used this system.

## AI-use declaration
I used an artificial intelligence assistant only to understand the lab code and theory in general terms and to check the code where I have to fill it out myself. And I wrote the conclusions myself.
All the conclusions contained in this document are my own: I checked each cell myself, read the laws myself, and wrote an analysis based on the results of my own analysis. I used Yandex Translate only to translate individual words and phrases (for example, legal terms from Kazakh/Russian into English) and not to create text.
