# JUSTIFICATION — the design of the harness, and why
<!-- STUDENT: replace -->

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

- What is the system for, stated narrowly? What decisions may it make on its own, and which may it only
  inform?
- For each control you built (input validation, blinding, refusal, log, escalation): which
  constraint made you build it, and where in the code can a reader find it?
- Did any constraint change the architecture itself, so that the design would be different without it?
  Name a decision you made differently because of a constraint, and what you would have done otherwise.

## 2. What it must refuse

(Dimension: Refusal Boundaries and Enforcement.)

- What does it refuse (purposes, inputs, applicants)? Where is each refusal enforced in code, and
  where in your submitted run did it fire?
- For each refusal: **which legitimate case does it also block**, and who bears that cost? Is that cost
  acceptable, and why?
- What happens to a refused request: is it recorded, is anyone told, can the person ask for a decision to
  be made by a human instead?

## 3. What record it keeps

(Dimension: Accountability Record and Auditability.)

- What is the **audit question** your record is designed to answer (for example, "why did this
  applicant get this outcome, and could someone else reconstruct it without re-running the model?")?
  State it in one sentence, then show that a record in `decisions.jsonl` answers it.
- What is in each record, and what is left out on purpose (for example, personal data you chose not to
  store)? Why that line?
- How long is the record kept, who may read it, who may change it, and what stops a record being
  edited after the fact? What does the record not let you reconstruct?

## 4. The legal constraints

Before the search:
Prediction: I think validate_input will be the hardest control to tie to a specific provision. It is a sanity check: is the GPA a number between 0 and 4, is the age an integer, is consent a real boolean. This looks like engineering hygiene rather than a regulated activity. I expect a data-protection law to say at most that data must be accurate or kept secure, and I do not expect any article to say "check that GPA is a float". So I expect to find only a general clause on accuracy or integrity of processing. That said, validate_input does more than check types: it refuses to make a decision on impossible values, so it keeps the system from deciding on garbage data. If the clause I find does not bind this prototype, I will mark it in the register as a benchmark.

(Dimension: Identification and Citation of Legal Constraints.)

- Which instruments apply to an automated screening decision about people, and why do they
  apply to this system (jurisdiction, subject matter, who is the operator)? Cite the instrument and the
  provision for each.
- For each, what does the law **require** of the system, and what does it merely **permit** or leave
  to you? Where did you do more than the law requires, and why?
- Where is the text you cite stated to apply to automated decisions specifically? If you could
  not find a provision for a control, say so rather than inventing one.

## 5. The ethical principles

(Dimension: Identification of Applicable Ethical Principles.)

- Which principles govern an automated screening decision, and which decision in **this** system does
  each bear on (not in general)? Cite the source in which you found each principle named, and say
  what status that text has.
- Where do the principles conflict with one another in your design (for example, between
  treating applicants identically and treating applicants fairly given what you measured)? Which did
  you put first, and why?
- Where does ethics demand something the law does not?

## 6. The trade-offs

(Dimension: Trade-off Analysis and Justification.)

- For each control: what does it cost (errors, speed, reviewer time, applicant privacy, extra model
  calls), and what harm does it prevent? Use your before-and-after measurement where you can.
- The width of your escalation band is a trade-off too: how many applicants does it send to a person, who reads
  them, and what happens to the cases outside it that nobody re-checks? A threshold calibrated on one
  configuration does not transfer to another: did you recalibrate it after you changed what the model
  sees, and what share was actually escalated?
- Why did you choose the balance you did, and what would make you choose differently?
- What does the design **still fail to protect against**? What would closing that gap take, and why did
  you not do it?

## AI-use declaration

Half a page: which AI tools you used in this assignment and for what. If you used none, say so.
