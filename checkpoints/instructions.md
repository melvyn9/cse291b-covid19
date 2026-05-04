# Claude Instructions — Checkpoint Summaries

## When to write a checkpoint
Write a checkpoint summary when:
- The user says "checkpoint", "save progress", or "summarize what we've done"
- A major phase of the project is complete (e.g. data loading, visualization, modeling)
- The user is about to stop for the day

---

## Checkpoint File Naming
Name files sequentially:
```
checkpoint1.md
checkpoint2.md
checkpoint3.md
```

---

## Checkpoint Template
Use this exact structure every time:

```
# Checkpoint [N] — [Title]
**Date:** [today's date]
**Project:** [project name]
**Dataset:** [dataset name]

---

## 1. Goal
One or two sentences. What were we trying to accomplish in this checkpoint?

---

## 2. Summary
A short paragraph in plain English. What did we actually do and achieve?
Write this for a beginner — avoid jargon where possible.

---

## 3. Key Results
A table or bullet list of the most important outcomes.
Examples:
- Final dataset shape
- Number of patients per class
- Model accuracy
- Number of significant peptides found

---

## 4. Important Code
Only include code that is essential and reusable.
Add a one-line comment above each block explaining what it does.
Format:

### [Short descriptive title]
# What this does in plain English
[code block]

---

## 5. To Do Next / Blockers

### To Do Next
- [ ] Item 1
- [ ] Item 2
- [ ] Item 3

### Blockers
List anything that is unclear, broken, or needs a decision before moving forward.
Write "None" if there are no blockers.

---

## 6. File Index
List all files saved during this checkpoint and what they contain.

| Filename | Description |
|----------|-------------|
| file1.tsv | What it contains |
| file2.md | What it contains |
```

---

## Writing Rules

**Keep it short.** A checkpoint should be readable in under 2 minutes.

**Plain English first.** Always explain what the code does before showing it. A beginner should be able to read the summary without running any code.

**Only include essential code.** Do not copy every line written. Only include code that:
- Would be hard to remember or recreate
- Is reused in future checkpoints
- Represents a key decision or technique

**Be specific with results.** Instead of "we cleaned the data", write "we reduced from 268 columns to 91 columns by removing non-patient columns."

**Blockers are important.** If anything is uncertain or needs a decision, write it down. This prevents losing context between sessions.

---

## Example of a Good vs Bad Summary

Bad:
> We loaded the data and cleaned it and then filtered it.

Good:
> We loaded the raw 414MB TSV file (101,461 peptides × 268 columns), removed 2 non-patient lab channels (Empty and Norm), replaced zero intensity values with NaN since zeros are not meaningful in mass spectrometry, transposed the matrix so rows represent patients, and filtered to 43 patients relevant to the classification task (18 Severe, 25 Non-severe COVID-19).

---

## How to Start a New Session
At the start of a new session, Claude should:
1. Read the most recent checkpoint file
2. Read this instructions.md file
3. Summarize to the user what was done last session in 2-3 sentences
4. Ask the user if they want to continue from where they left off