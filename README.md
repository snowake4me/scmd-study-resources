# SCMD / DEX-401 Study Resources

A bespoke collection of study materials built around my own exam journey — not a generic course summary. These resources are generated with Claude and curated by me. Your mileage may vary.

---

## The Arc

First things first - I thought I'd see where I was at fresh from a week in DEX-401, prior to any additional study.

To take the practice exam I used a custom HTML answer sheet rather than pen and paper — it handles timing, flagging, and automatic scoring with a per-section breakdown:

**[scmd_practice_exam_answer_sheet.html](scmd_practice_exam_answer_sheet.html)**
Open in any browser. Enter answers as you go, flag questions for review, then hit Score to see your result and where you lost points by section. The answer key and section mappings are baked in — no exam question text is included.

Results: EPIC fail (60%)

### Round 1 — Building from Missed Questions

The first generation of materials came directly from practice exam analysis. Rather than re-reading course content wholesale, I identified every question I missed, diagnosed the underlying gap, and had Claude synthesize targeted study content from those specific failure modes.

That process produced two documents:

**[SCMD_DEX401_Complete_Reference_v2.md](SCMD_DEX401_Complete_Reference_v2.md)**
A full 12-section exam reference covering all topics with exam weights, key rules, common traps, and the distinctions that matter on the actual test. Heavy on tables and decision rules — built to support active recall, not passive reading.

At this point, I walked through each of the "Knowledge Check Quizzes" for each module from the DEX-401 courseware. Capturing my errors on those quizzes drove a further round of targeted refinement, producing:

**[SCMD_FinalGapDrillGuide_v2.md](SCMD_FinalGapDrillGuide_v2.md)**
The pre-exam drill guide, updated the evening before the test with empirically verified behavior from live Anypoint Studio testing. Focused exclusively on weak spots: the Mule Event model, error handling chains, connector payload behavior, DataWeave syntax gotchas, and batch/scatter-gather mechanics.

**[SCMD_Persistent_Failures_CheatSheet.md](SCMD_Persistent_Failures_CheatSheet.md)**
The exam-morning cheat sheet. Every concept that tripped me up across multiple practice sessions, with hit counts, the exact trap, and the rule to overwrite it.

### The Exam

April 9, 2026. Did not pass.  Had to wait several agonizing hours for results via email - and didn't actually get a final score - just a breakdown of my performance by topic.  I was strong, in a couple (80%) - but weak (50-60%) in a lot more - and actually had one 0%.  Hope that was a small topic!  Lots of work to do - and yet, I STILL want to strike while the iron is hot and retake this **soon**.

---

### Round 2 — Preparing for the Retest

Post-exam topic-by-topic reflection. What actually showed up, what the drill guide got right, where the real gaps were. Followed by a new iteration with Focus on Force — notes on their approach, what it surfaces differently, and how it informs the next round of targeted prep.

After REALLY digging in via Focus on Force - with both deep dives on each discrete topic, as well as additional practice exams - I was chasing down each gap and working hard to fill it with a clear mental model.  Through iteration, flagging questions where I wasn't confident of the answer, and really digging into nuanced flow and error handling behavior, edge cases, and even 'trick questions' -- I started to feel a bit more confident.  After 'barely passing' the first practice exam - I scored pretty well on the 2nd.  At that point, I was down to just a handful of questions I flagged - and go figure, all but one of the questions I missed - I had flagged.  Time to REALLY drive those details home.

---

### Round 3 — Hands-On Lab

The part that was missing the first time: actually building something.

In addition to deploying the DEX-401 student project files, I built a hands-on lab project from scratch to force real fluency — not recognition, but execution. The work includes stepping through complete API flows, watching how payloads evolve through each processor, tracing error propagation across flow boundaries, and writing real DataWeave transformations under realistic conditions.

Project files and Postman collections will be added here.

---

## How to Use This

These aren't meant to replace the official Trailhead or Focus on Force materials — they're a surgical overlay. The most useful way to approach them:

1. Use the **Complete Reference** as a structured review of what's on the exam and how it's tested
2. Use the **Drill Guide** for active recall on the high-risk topics (error handling and event model especially)
3. Use the **Cheat Sheet** as a final sanity check before sitting down — these are the exact traps that have bit me repeatedly

If you're preparing for the same exam, the specific failure patterns documented here are worth studying regardless of your own practice results. Error handling chains, scope boundary behavior, and Batch Job semantics are reliably tricky across all versions of this exam.

Better still - use this repo as an example - and build your own, tailored version.
