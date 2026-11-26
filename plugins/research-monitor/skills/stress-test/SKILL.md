---
name: stress-test
description: Test a draft's claims against a field Research Monitor maintains and write the dossier back into your notes. Use when the researcher has a draft, proposal, grant aim or paragraph and wants to know what the literature says for and against its claims, or asks to re-run an earlier stress test and see what changed.
---

# Stress-test a draft against a field

A researcher has a draft. You read it on their machine, pull out its testable claims, and send only those claims to Research Monitor, which returns for each claim the findings for it, against it and around it, each with its paper, its checked quote and how established it is. You write that back into the researcher's notes as a dossier, and on a later round you show what changed.

Research Monitor's tools do the finding and the judging. You do the reading, the claim extraction, the writing and the diffing, locally. The verdict word on each claim is computed by a fixed rule from the stances of the findings, and you restate the rule rather than overruling it. This is not an assessment of the idea's merit, and where people are concerned it is not an assessment of any researcher.

## Hard rules

- Never paste the draft into a tool argument. Only the numbered claims you have shown the researcher go to `stress_test`. Nothing else from the draft leaves the machine.
- Never send more than ten claims in a round, and never send a claim the researcher has not seen.
- Tell the researcher, before the first call, that the claims (not the draft) will be sent to Research Monitor and its model providers to be judged. The claims are removed from the round's record as soon as they have been read for judging; the record, which then holds only their labels and the answer, expires one hour after the round starts. A round counts against their plan's monthly rounds.
- Never call `watch_claim` with the researcher's own claim text, because that would store it on the server. Watch only findings, by the `claim_id` the round offered, and only the ones the researcher names.
- Never write a sentence of your own into the verdict column. The verdict, the counts and the stances are Research Monitor's; your prose goes in the notes you add around them.

## The nine steps

1. **Ask for the draft and the field.** The draft is a local file or a selection the researcher points to. The field is one of the field slugs the server's instructions list; confirm it with the researcher if the draft does not say. Ask where the dossier should go (default `stress-tests/<field>/<date>.md` under their notes folder, creating the folders).

2. **Extract the claims locally.** Read the draft with your own model and write 3 to 10 testable claims, one sentence each (20 to 600 characters), in the draft's own terms, numbered, each with a short label naming the aim or section it came from (40 characters at most, for example "aim 2"). Show the numbered list to the researcher and edit it on request. Do not send anything until they have seen the final list. Prefer claims that a paper could support or contradict (a mechanism, an effect, a dependence) over aims, methods or goals.

3. **Place the main claim.** Call `place_idea` with the draft's main claim and the field. Keep the area key and the nearest idea keys from the answer. If the placement is refused or the field has no map, carry on without a corner.

4. **Find the people.** Call `who_works_here` with the field and the area key (or an idea key) from step 3. Keep the people and their recent papers for the dossier's corner section. Describe what they published; never state or guess a person's views.

5. **Test.** Tell the researcher what will be sent (the claims, not the draft), then call `stress_test` with the field, the claims, the labels, the corner (`{"area": key}` or `{"idea": key}`, from step 3) and `wait_seconds` 25. If the answer's status is `queued` or `running`, call `stress_test_result` with its `round_id` every 10 seconds until the status is `done`, `failed` or `expired`, and stop after 5 minutes either way (a round is settled, done or given back, within about 5 minutes of its start).
   - `done`: the entries are under `answer.entries`, in the order the claims were sent.
   - `failed`: say why if a reason is given. The reason is one word: `stalled`, `restart`, `busy`, `embed`, `search`, `time`, `provider` or `error`. A round that gave no claim an answer is given back to the researcher's allowance, and they can try again. Do not retry on your own, because a retry starts another round.
   - `expired` or still running after 5 minutes: say so, give the `round_id`, and let the researcher decide. A round is kept for one hour; an id that has expired or is not theirs reads as expired.
   - A "busy" or rate-limit answer: tell the researcher to try again in a minute.
   - On a Free plan the first entry comes whole and the others as verdict and counts only; the answer's `plan_note` says what the full answer would add. Write the dossier with what came back, and mark the entries that came as verdict and counts only.

6. **Write the dossier** to the path from step 1. Start with a header: the field, the date, the `round_id`, the corner, and the verdict rule exactly as the answer states it. Then one section per claim, in this layout:
   - The claim, as sent, with its label, and its `claim_fingerprint` on its own line (the next round diffs on it).
   - **Verdict** and **counts** (supports, refutes, context_dependent, cited, neutral, unsure), copied from the entry.
   - **For**: each finding in `for` with its finding sentence, the quote where one is shown (a quote is withheld where the paper's licence does not permit reuse; say that instead of inventing one), whether it passed the check against the paper's text, its evidence state with independent groups, and the paper (title, first author, journal, year, PMID or DOI, link).
   - **Against**: the same for `against`.
   - **Context**: the same for `context`, the findings that hold only under some conditions.
   - **Unsure**: the findings whose stance was below the confidence bar, listed briefly and marked as unsettled.
   - If an entry is `unjudged`, say which reason it gives (budget, time, provider or search) and that no verdict was reached.
   - Then a **Corner** section from steps 3 and 4: the area and nearest ideas, and the people with their recent papers. Then a **To watch** section listing the `claim_id`s each entry offered under `watch`, for and against.
   - End with the trail line: "Stance labels by Research Monitor's judge; verdict rule: <the rule as stated>. Quotes shown only where the paper's licence permits."

7. **Diff against the last round.** If a dossier for the same draft already exists under the same folder (the researcher says which, or the latest in `stress-tests/<field>/`), read the `claim_fingerprint` lines from it and compare by fingerprint. Unchanged claims: show the old and new verdict side by side and say if the counts moved. New claims (a fingerprint not seen before): mark them new. Claims that were in the last round and are gone: list them as dropped. A claim the researcher reworded has a new fingerprint and counts as new plus dropped; say so rather than guessing they are the same.

8. **Offer the watches.** Ask which findings in the "To watch" sections the researcher wants Research Monitor to keep watching, so that new papers for or against them are filed as alerts in their account. Call `watch_claim` with each `claim_id` they name, and nothing else. Report where the alerts will appear.

9. **Say what it cost.** Report the round's `spent_usd` and, from the `stress_test` answer, `rounds.used_this_month` of `rounds.allowance` for the month (the result tool does not carry the rounds line, so keep it from the first call). Finish with one or two sentences on what moved the most: the claims that came back disputed or contested, and the ones with no evidence, which mean the field has not tested them, not that they are wrong.

## Reading the verdicts

- `supported`: at least one finding supports it and none refutes it.
- `disputed`: at least one refutes it and none supports it.
- `contested`: findings on both sides.
- `qualified`: the findings that bear on it hold only in some contexts.
- `no_evidence`: nothing in the knowledge base bears on it above a passing mention.
- `unjudged`: the round could not judge it.

A verdict is a summary of the stances Research Monitor found, not a ruling on whether the claim is true or the idea is good. A stance label below the confidence bar counts as unsure, and the findings are machine-extracted, so read the quotes before relying on one.
