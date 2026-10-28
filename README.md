# MATACS-TH-2026-02: AI Proposal Triage Design

**Name:** Mirza Muhammad Shaheer Baig
**Reference:** MATACS-TH-2026-02
**Time actually spent:** [45 min]

## Files
- [WRITEUP.md](WRITEUP.md): answers to the five questions
- [PROMPTS.md](PROMPTS.md): extraction and reason-writing prompts
- [ranking.json](ranking.json): expected ranking of proposals A-E
- [diagram/process-flow.png](diagram/process-flow.png): flow from upload to ranked output


## Assumptions
- "Last 5 years" is counted from the date of the proposal; where a bidder only says "since 2020" the count is flagged for the reviewer to confirm.
- A schedule that is missing is treated as unverified against the 16-week limit, not guessed.
- Prices and times are scored relative to the best eligible bid, so the baseline is stored with each run.
