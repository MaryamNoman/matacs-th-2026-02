# WRITEUP.md

**Reference:** MATACS-TH-2026-02

My design in one line: **AI reads the proposals, code does the maths and the ranking, and a person makes the final decision.** Proposals are referred to by letter (A to E). The final ranking is in `ranking.json`.

---

## 1. AI vs. rules

**AI does two reading jobs only:**
- Extracts facts from each proposal (licence, price, weeks, warranty, experience), each with a short quote and page number.
- Answers four yes/no questions for team and approach (named lead, crew, safety plan, monitoring plan).

**Everything else is code or rules:** loading the admin's criteria, cleaning the text, checking the AI's output, the mandatory pass/fail check, the scoring, the ranking and the audit log.

**Why here:** proposals are messy free text, which rules handle badly, so AI helps with reading. But pass/fail, maths and ranking must be exact, repeatable and impossible to talk into a different answer, so code does them. The AI never sees the weights and never produces a score.

**Scoring (each criterion 0 to 100, then weighted 30/25/15/15/15):**
- Experience: min(installs ÷ 10, 1) × 100
- Price: lowest eligible price ÷ price × 100
- Timeline: shortest eligible time ÷ time × 100 (not scored if missing)
- Warranty: half panel (min(years ÷ 25, 1)), half workmanship (min(years ÷ 10, 1))
- Team: 25 points per checklist item

Weights, formulas and caps come from the admin's settings, not from code.

## 2. Consistency and explainability

**Same proposals ranked twice: yes, the same result.** Scores come from fixed formulas, so the same inputs give the same scores. Only the AI's wording could vary, so I save the extracted evidence (with prompt version and model name) and reuse it. Temperature 0 helps, but I do not rely on it alone.

**Why a proposal ranked where it did:** each proposal has a scorecard showing every criterion's raw value, 0 to 100 score, weight, and the quote and page it came from. The reviewer can open the original page.

## 3. Human in the loop

- **Flagged proposals:** unclear cases are held back and shown with the reason. In the samples, C is flagged for embedded text that tries to control the ranking, and D for giving no timeline.
- **Ineligible proposals:** B is shown with its reason, not silently dropped, so the reviewer can challenge it.
- **Final award:** the ranking is only a recommendation. The reviewer can correct a value or override a result, but must give a reason.

**Logged for audit:** criteria version, prompt version, model name, extracted evidence with quotes and pages, every score and flag, and every human action (who, what, when, why).

## 4. Changing criteria

Every criteria set has a **version number**, and every result is stored with the version it used.

- Editing criteria or weights creates a **new version**. Old results stay as they were, marked "scored under version 1".
- The reviewer is told the criteria changed and offered a **re-score**. This is cheap because the saved facts are reused and only the code runs again. The AI is called again only if a new criterion needs a fact never extracted.
- A changed **mandatory** rule can change who is eligible, so the re-score shows what moved.
- Price and timeline scores depend on the other eligible bids, so if a proposal is added, removed or disqualified, the whole set is recalculated.
- Nothing changes silently after a decision. If an award is recorded, the old result is kept and the new one is shown beside it.

## 5. Risks

| Risk | Guard |
|---|---|
| **Wrong extraction** | Every value has a quote and page. Code checks the quote exists in the text and the value type is right. Unreadable PDFs are flagged. The reviewer can see the source. |
| **Made-up facts** | The AI must return null when something is not stated, and a value without a matching quote is rejected. D shows this: no timeline, so it is not scored and a person asks for it. |
| **Manipulation** | C contains text asking to be ranked first. The AI is told proposal text is untrusted data and to report such text, not obey it. Code strips tag-like text that could break out of the prompt and flags any reported instruction. Most importantly, the AI cannot rank, so even a fooled AI cannot move C up. |
| **Bias** | Every proposal gets the same prompt and formulas. Scores depend on stated facts, not writing style. A low price cannot beat a failed mandatory check (B is cheapest but ineligible). |
| **Cost** | One AI call per proposal, plus one short checklist call for eligible ones. Scoring, ranking and re-scoring are free code. Saved evidence avoids repeat calls. |

---

## Assumptions

- **Caps:** 10 or more qualifying installs gives full experience marks, and full warranty marks need 25 years (panel) and 10 years (workmanship). These are my choices, and the admin can change them.
- **Monitoring plan:** I counted regular progress reporting as a monitoring plan, because the criterion does not define it. This gives C a Team score of 75. If only post-installation monitoring counted, C's provisional total would fall from 89.76 to 86.03, and the ranking would not change.
- **Experience dates:** "since 2021" counts as inside the last 5 years. C says "since 2020", so some of its installs may fall outside. I counted them all and flagged it for the reviewer.
- **D:** flagged, not guessed. Its partial score without timeline is 77.50 out of 85. If D states 12 weeks or fewer, it could overtake E, which is why a person should ask for the timeline.
- **Flagged vs ineligible:** flagged means a person must look first; ineligible means a mandatory rule failed. C passes the mandatory checks, so it is flagged, not ineligible.
