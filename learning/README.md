# Learning material

One folder per topic, `learning/T01/` to `learning/T18/`, created when that topic starts. Every
topic folder uses the same layout so progress can be compared across topics. Product code never
lives here; it is organised by product structure elsewhere in the repository, and the topic folder
links to it.

## Standard layout of `learning/Txx/`

```text
learning/Txx/
  README.md
  01-concepts.md
  02-labs.md
  03-self-check.md
  04-build-task.md
  05-review-checklist.md
  answers/
    self-check-answers.md
  notes.md
  SUMMARY.md
```

| File | Purpose | Blueprint card field |
|---|---|---|
| `README.md` | Topic name, depth (OWN / STEER / AWARE), size, why the manager needs it, dependencies, capstone steps it contributes to, and current status | 1, 2, 10, 11 |
| `01-concepts.md` | What must be understood, what can be skipped or delegated, and native-versus-build reasoning, with links to official docs. Unverified vendor facts are labelled UNVERIFIED | 3, 4, 5 |
| `02-labs.md` | Small hands-on experiments that demonstrate the concepts (for T01, the probes in `probes/`). Each lab states what it shows and what it does not | 3, 5 |
| `03-self-check.md` | The understanding-check questions and the required drawing or explanation. Questions only | 8 |
| `04-build-task.md` | The build deliverable, the decisions it unlocks, acceptance criteria, and links to where the code lives in the product tree. For OWN topics it also states the independent exercise, which the learner writes without the solution being provided | 6, 7 |
| `05-review-checklist.md` | The review checklist for generated code, expanded with the relevant R6 items (G1-G10) and how each is detected | 9 |
| `answers/self-check-answers.md` | Answers to `03-self-check.md`, kept separate so the questions can be attempted first | 8 |
| `notes.md` | Working notes, open questions and surprises found while studying | — |
| `SUMMARY.md` | Written when the topic finishes: what was learned, what was built, decisions taken (with `docs/decisions.md` IDs) and remaining gaps | — |

## Rules

- For the OWN topics (T02, T05, T07, T08, T11, T14), the independent exercise in
  `04-build-task.md` is done by the learner. Assistance is limited to hints and review.
- A topic is `done` in `docs/progress.md` only when `SUMMARY.md` exists and the self-check has been
  answered.
