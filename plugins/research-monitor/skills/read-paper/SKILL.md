---
name: read-paper
description: Read a PDF paper locally on the researcher's model, place its claims against Research Monitor's maintained knowledge base and the researcher's own projects and watched ideas, and optionally save it into their Brought in list. Use when the researcher attaches or points to a PDF, preprint, or manuscript and wants a grounded critique or lab brief.
---

# Read and place a paper with Research Monitor

A researcher has a PDF or manuscript. Your own model reads the paper locally, extracts its metadata and core testable claims, and sends only those structured claims and identifiers to Research Monitor. Research Monitor runs zero text-model inference on your PDF: it checks publication integrity (retractions and corrections), places the claims on the maintained field map and checked knowledge base, and matches them against the researcher's own projects, hypotheses, and watched claims. You then synthesize the grounded critique and lab brief locally.

## Hard rules

- Never paste raw PDF text, full sections, or a manuscript body into any tool argument. Send only the paper's title, optional DOI or PMID, optional gene or pathway terms, and 1 to 8 concise one-sentence claims (20 to 600 characters each) extracted locally.
- Before calling `place_paper`, show the researcher the extracted title, identifiers, and claim list and note that only those strings (never the PDF or manuscript body) are sent to Research Monitor and its embedding provider to be matched and are not kept. For an unpublished manuscript or confidential draft, wait for the researcher's go-ahead before sending.
- `place_paper` is read-only and stores nothing; `save_brought_paper` and `watch_claim` write to the researcher's Research Monitor account and always require explicit confirmation first.
- Never invent or reconstruct a quote that Research Monitor withheld for licence reasons. State plainly when a supporting passage is withheld at the source.
- Separate what the PDF claims from what Research Monitor's checked knowledge base shows, and flag any retraction, correction, or expression of concern prominently at the top.

## Workflow

1. **Read the paper locally.** Inspect the PDF or manuscript on the researcher's model and pull out:
   - Paper title, plus DOI and PMID if printed on the paper.
   - 3 to 8 testable one-sentence claims (20 to 600 characters each) covering the core findings and mechanisms.
   - Key genes, pathways, cell types, or methods (up to 15 short terms).
   - Methods summary, experimental controls run, and candidate missing controls or caveats noticed while reading.
   - Show the extracted title and claims to the researcher before calling `place_paper` (and wait for approval if the paper is an unpublished manuscript).

2. **Place the paper in Research Monitor (`place_paper`).** Send `{ title, doi?, pmid?, topic?, claims, genes_pathways? }` to `place_paper`.
   - Specify `topic` when you know which maintained field fits best (`gut_biology`, `cancer_cell_therapy`, `foundation_models`, or a field the account follows). When `topic` is omitted, `place_paper` uses the account's first followed field; if the account follows no field, `field_placement` returns a `note` listing available slugs.
   - Inspect `in_library`: if `in_library.found` is `true`, check `in_library.integrity_status` for any retraction, correction, or expression of concern and note whether Research Monitor already holds a deep read (`in_library.has_deep_read`). If `in_library.found` is `false`, note plainly that the paper is not yet in the account's visible Research Monitor library.
   - Inspect `field_placement`: if it returns a `note` without `area` or `findings` (for example when a field map is not built yet or placement is temporarily unavailable), say so plainly and continue with `in_library`, `your_ecosystem`, and your local critique. Otherwise inspect `field_placement.area`, `field_placement.ideas`, `field_placement.findings` (each already carries `passed_check`, `evidence_state`, `independent_groups`, `support_papers`, `contradiction_groups`, and licence-safe `quote`), `field_placement.open_questions`, and `field_placement.scales` (area/idea similarities use the document scale and findings use the query scale, so do not compare the two numbers directly).
   - Inspect `your_ecosystem` for overlaps with the researcher's active `projects`, `watched_claims`, and `kb_connections`.
   - If the answer includes a `plan_note`, state plainly what came back as a preview and surface its link without adding promotional language.

3. **Deepen context when helpful.**
   - `field_placement.findings[]` already include `evidence_state` and `independent_groups`. Only call `how_established` with a finding's `claim_id` when the researcher wants the full proposition breakdown across groups and model systems; note that `how_established` returns `PLAN_REQUIRED` on a Free account.
   - To see which labs publish in that corner, call `who_works_here` with `topic = field_placement.topic` and EITHER `area = field_placement.area.key` OR `idea = field_placement.ideas[i].idea_key` (never both `area` and `idea` in the same call). Describe what each author published and never guess a person's views.

4. **Synthesize the grounded critique locally.** Present a crisp, researcher-grade readout:
   - **Publication status** (only if `in_library.integrity_status` is non-null, placed first).
   - **Bottom line** (2 to 3 sentences in plain scientific English).
   - **Where it lands in the field** (map area, nearest ideas, and checked supporting or conflicting findings with PMID or DOI citations and check status).
   - **Bearing on your work** (matched projects, hypotheses, watched claims, and field KB concepts).
   - **Methods & critical caveats** (controls run, missing controls, and limitations read locally from the PDF).

5. **Offer persistent follow-up (opt-in only).**
   - If the paper has a DOI or PubMed ID, ask if the researcher wants to save it and its extracted claims into their Research Monitor `Papers -> Brought in` list (`save_brought_paper`) so it stays linked in their account without uploading the PDF. `save_brought_paper` needs one of the two and takes the paper's record from PubMed or OpenAlex, never from the title you send; a manuscript with neither can be placed but not saved, so say that plainly and do not offer to save it.
   - If they confirm and call `save_brought_paper`, relay the returned `url` (reader page) and `where` (`Papers -> Brought in`) links.
   - If `save_brought_paper` returns `PLAN_REQUIRED`, relay the message and informational link plainly; if it returns `403 insufficient_scope`, tell them to allow write access when prompted to reconnect; if it returns `RATE_LIMITED`, note that today's brought-in paper limit was reached and suggest trying again tomorrow.
   - Ask if they want to keep watching any specific finding from the placement (`watch_claim` with the finding's `claim_id`).
