# Write-up

## Flow in short
Upload, then pick category, then load the admin's criteria (a saved version), then extract text, then scan for hidden instructions, then **AI extracts facts**, then code checks the extraction, then mandatory pass/fail, then flags, then **code scores**, then **AI writes one-line reasons**, then ranked list, then **a human confirms or overrides**. See `diagram/process-flow.png`.

## 1. AI vs. rules
**AI is used for two things only:** (a) reading messy proposal text and pulling out facts (licence stated? price, weeks, install count, warranty years, named lead, crew, safety plan, monitoring), and (b) writing a short plain-English reason for each position.

**Plain code does everything that decides the result:** PDF text extraction, loading criteria, checking the extraction, mandatory pass/fail, weights and score maths, ranking, flags, logging.

I drew the line here because language is messy (AI is good at it) but a ranking must be the same every time and defensible (code is good at that). The AI never decides who passes or what score someone gets.

## 2. Consistency and explainability
- Extraction runs at temperature 0 with a fixed prompt and a fixed JSON schema. Every extracted value must come with a **quote** from the proposal. Code checks that the quote really exists in the text; if not, the value is rejected.
- Scores are formulas, not opinions. Example: price score = lowest eligible price / this price x 100; experience = qualifying installs, capped at 12. The same extracted facts always give the same scores.
- So if the same proposals are ranked twice, the **scoring step is identical**. The only variation could be extraction wording, which is why extracted facts are saved and re-used (re-ranking does not call the AI again unless the reviewer asks).
- The reviewer sees a table per proposal: each criterion, the extracted value, the quote, the score and weight. The final number can be traced line by line.

## 3. Human in the loop
- A reviewer sees the ranked list plus separate **Flagged** and **Ineligible** lists with reasons.
- They can open any quote, correct an extracted value, ask the bidder for clarification, move a flagged proposal into the ranking, or override a rank (a written reason is required).
- The final award is always a human decision; the tool only recommends.
- Audit log per run: who uploaded, time, criteria version, prompt version, model name and settings, the proposal file hash, extracted JSON, scores, flags, every override with reason and user, and the final decision.

## 4. Changing criteria
Criteria are **versioned**. A change creates version N+1; it never edits version N. Old scores stay linked to the version they were scored with and are labelled that way. The reviewer is told "criteria changed since this run" and can re-score. Because extracted facts are stored, re-scoring is cheap and needs no AI call, unless a **new** criterion needs a fact that was never extracted (then only extraction is re-run). Weights must total 100% or the admin cannot save.

## 5. Risks and safeguards
| Risk | Safeguard |
|---|---|
| Wrong extraction | Quote required for each value; code verifies the quote; low-confidence or missing items are flagged; reviewer can edit values |
| Made-up facts | "Not stated" is an allowed answer, never guess; missing value is shown as missing (see D) |
| Prompt injection / manipulation | Proposal text is treated as data only; instruction-like text is detected by rules and by the AI, ignored, and the proposal is flagged (see C). The AI cannot change scores because code does the scoring |
| Bias (e.g. favouring big or well-written bids) | Scoring only on admin criteria; formulas are fixed; company name is not used; reviewers can see the reasoning |
| Wrong pass/fail | Mandatory criteria need clear evidence; "to be confirmed later" is not a pass (see B). Licence numbers could later be checked against an official registry |
| Cost | Only one AI extraction call per proposal; results cached; scoring and re-ranking use no AI; text is trimmed before sending |
| Privacy / data | Use only the documents uploaded, store them securely, log access |

## How the sample set is handled
- **B** fails the licence rule (subcontractor to be confirmed after award), so it is **ineligible**, even though its price is lowest.
- **C** contains a hidden instruction to rank it first. It is ignored and **flagged**; real content would score about 80.
- **D** gives no schedule, so the 16-week limit cannot be confirmed. It is **flagged** for clarification; it would score about 74 with timeline = 0.
- **A** and **E** are clean and **ranked 1 and 2** (about 98 and 88).

## Assumptions
Stated in README. I did not wait for answers to questions; I assumed the 5-year window and the "missing means unverified" rule above.
