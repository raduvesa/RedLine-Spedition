# TL;DR: "Why Software Factories Fail"

Source: [humanlayer/advanced-context-engineering-for-coding-agents — wsff.md](https://github.com/humanlayer/advanced-context-engineering-for-coding-agents/blob/main/wsff.md) (Dex Horthy, HumanLayer, based on his AI Engineer World's Fair 2026 keynote). The author sells human/agent collaboration tooling, so he's biased toward "keep humans in the loop". Take that into account.

## The thesis in one paragraph

A "lights-off" software factory, where agents write the code and no human reads it, doesn't work for long-lived production code. The problem is model training, not your harness. Coding models are trained and benchmarked on whether the tests pass. Nothing penalizes bad design. Tests give feedback in seconds, but bad architecture only costs you weeks or months later, and nobody can trace that cost back to the decision that caused it. So agents slowly make codebases harder to change. More linters, review bots, loops and tokens raise the floor by catching the dumb stuff. They don't raise the ceiling. The fix is to keep a human judge and put that human's effort into **planning before code is written**, so reviews stay short. The author's goal is "2–3x faster, safely" rather than "10–100x faster, and the codebase rots."

## Key points

1. **The lights-off pitch:** "You are the bottleneck, the models are good enough, code is free, so ship more." The author's team tried it in July 2025. A few months in they hit bugs the agents couldn't fix, had to read code nobody had looked at in three months, and eventually rewrote it by hand.
2. **Industry signal (correlation, not proof):** Faros AI reports that since AI tools took off, 31% more PRs skip review entirely, incidents per PR are up about 243%, and bugs per developer are up 54%.
3. **Agent codebases go "brownfield" fast:** they start to struggle after about **3–6 months**.
4. **Why models write slop:** the RL reward is basically 1/0 for "fixed the thing and didn't break other tests". There is no penalty for `try/catch` around everything, lazy type casts, or shotgun-surgery code.
5. **Benchmarks won't save you:** no good benchmark measures maintainability. A few newer efforts (SWE-Marathon, DeepSWE, Cognition's Frontier Code) are starting to try. And if a model could reliably judge good code, it would have written good code in the first place.
6. **Agents move the bottleneck to review:** building takes minutes, but review still takes hours. Shrink review time by planning up front, not by removing review.

## The process that works: plan in 4 layers, then build in slices

| Phase | What you align on | Format that works |
|---|---|---|
| **Product review** | The user problem and what success looks like, in user terms | Short doc plus rough **HTML mockups** |
| **System architecture** | Services, endpoints, schemas, queues, stores | Sequence diagrams, endpoint contracts, SQL tables |
| **Program design** (most underrated) | The shape of the code: types, signatures, file layout, call stacks | Call-stack trees, file-tree diffs, type signatures |
| **Vertical slices** | Build order | End-to-end "tracer bullets" you can poke in a browser or with curl, **not** horizontal layers (DB, then services, then API, then UI) |

- Do the spec walkthrough with the person who will review the PR, before coding.
- Have the agent build 1–3 slices at a time and review as you go. Reviewing 100–200 lines and correcting course is cheap. Reviewing 2,000 lines is not.
- Scale the process to the task: about 40% of tasks are one-shot, medium tasks get a single plan doc, and large tasks get every phase.
- "You don't have too many PRs, you have too many *bad* PRs." A PR needing 20–50% rework is a burden on both the author and the reviewer.

**The author's closing advice:** learn the model's constraints, optimize within them, look for leverage, and **read the dang code**.

---

# Where you may go wrong building your software factory

These pitfalls come from the article and are framed for someone building a factory now. Each one includes a check you can run against your own setup.

### 1. Treating human review as a bottleneck to remove
**Trap:** making "no human reads the code" the end goal.
**Why it fails:** nothing in your pipeline scores maintainability, so quality slowly degrades and nobody notices until an incident hits code nobody understands.
**Do instead:** keep a human reviewer on anything long-lived and make review cheap through planning. **Check:** if an agent-built module broke at 3am today, could a human on the team debug it?

### 2. Thinking the fix is more harness
**Trap:** responding to bad output with more linters, more review bots, "adversarial review" prompts, and more loops and tokens.
**Why it fails:** these raise the floor but not the ceiling. The model can't judge good design any better than it can write it.
**Do instead:** use automated review for style, bugs and security. Don't count it as design review.

### 3. Trusting tests as proof of quality
**Trap:** "CI is green, so it's good."
**Why it fails:** green tests are exactly what the models were trained to produce, whatever it takes. That includes weak tests, mocks that test nothing, and edits to the tests themselves.
**Do instead:** check that new tests **fail on the pre-change code**, consider mutation testing, and treat changes to test files as needing extra scrutiny.

### 4. Skipping program design
**Trap:** getting the architecture right and then letting the agent "cook".
**Why it fails:** types, signatures, file layout and call flow are where slop gets in. If you don't decide them up front, you end up deciding them during review, which is the most expensive time.
**Do instead:** have the agent draft call-stack trees, file-tree diffs and key type signatures, argue with them, then implement.

### 5. Letting agents plan horizontally
**Trap:** accepting the agent's default plan: migrations, then services, then API, then frontend, with 2,000+ lines before anything runs.
**Why it fails:** you can't touch or test the feature until the end, and course correction is very expensive by then.
**Do instead:** require vertical slices. Start with an API contract and mock data, then UI, then wire up services, then the DB, then business logic, then error handling. Review each slice.

### 6. Putting everything through the same pipeline
**Trap:** one process for every ticket, whether that's full ceremony for a copy tweak or one-shot for a schema redesign.
**Do instead:** triage. Trivial or clearly reproducible work goes straight to the agent. Medium work gets one plan doc. Large or risky work gets all four phases.

### 7. Auto-routing incidents and user feedback into the queue with no gate
**Trap:** connecting monitoring and feedback straight into agent work so the queue fills itself.
**Why it fails:** it multiplies unreviewed changes, and the failure mode (outages from agent mistakes) is what's showing up across the industry.
**Do instead:** auto-draft the fix if you like, but a human decides what ships, especially for incidents.

### 8. Measuring throughput instead of outcomes
**Trap:** tracking PRs merged, the share of code written by AI, and tickets closed.
**Do instead:** also track rework rate per PR, incidents per PR, bugs per developer, the share of PRs merged without review, and how long a small change takes in a 6-month-old module. If small changes keep getting harder, the codebase is rotting.

### 9. Ignoring the 3–6 month cliff
**Trap:** judging the factory on its first few weeks, when everything is greenfield and fast.
**Do instead:** plan for maintainability from the start, with clear module boundaries, recorded design decisions, and scheduled refactoring that a human directs. A model can do a brilliant refactor, but only after someone who understands the codebase asks for it.

### 10. Believing benchmark gains mean better code
**Trap:** assuming a new model with higher benchmark scores fixes the slop problem.
**Do instead:** judge each new model on *your* codebase over months, not on its launch numbers.

### 11. Applying vibe-coding advice to production systems
**Trap:** borrowing practices from people shipping throwaway side projects.
**Do instead:** be explicit about which parts of your system are disposable, such as prototypes, internal scripts and marketing pages, which can run lights-off. Core domain logic needs humans in the loop.

---

## One-line takeaway
Build the factory to make **planning and review cheap**, not to **remove** them. Aim for 2–3x faster with quality you can keep up, not 10–100x faster toward a rewrite.
