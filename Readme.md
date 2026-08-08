# The 6-Beat Method - Cheat Sheet

A one-page driver's guide for the live coding-agent portion. **Print this.** The columns: what each beat *is for* (the signal), the prompt you send the **agent**, and the line you say **out loud to the interviewers** at that transition. The tempo — one beat at a time, you reviewing between each — *is* the interview.

> **Two rules that override everything (from real interviewer feedback):**
> 1. **Verify continuously, not at the end.** Beats 2–4 are a *loop per function*, not one pass. Never let untested code pile up — see the loop diagram below.
> 2. **Communication is graded.** Every beat has a spoken line *before* you act, not after. "I'm about to...", never "I just...". The **Say out loud** column is not optional flavor — it's a scored deliverable.

## The table

| # | Beat | What you're signaling | Prompt to the **agent** | Say **out loud** (before acting) |
| :--- | :--- | :--- | :--- | :--- |
| **0** | **Scope (humans, no agent)** | You scope before you code; you spot the design fork | *(no agent yet)* | "Let me restate: `<problem>`. Constraints that change the design: `<size / finite-vs-stream / single-vs-distributed / exact-vs-approx>`. My approach is `<structure>` because `<reason>`; the alternative `<X>` is valid too but `<why not>`. I'll have the agent plan it independently to check me." |
| **1** | **Plan, not code** | You extract the plan cheaply before expensive code | "I'm implementing `<problem + constraints>`. Before writing any code, tell me: the data structures, the algorithm, **the time and space complexity in terms of `<N, K, ...>`**, and the one main failure mode of your approach. Do **not** write code yet." | "I'm asking for its plan first — I want to catch a wrong approach for the price of a sentence, before any code exists." |
| **1.5** | **Correct if it drifts** (only if needed) | You redirect rather than follow the AI | "That `<naive plan>` is `<why it's wrong / doesn't constraint>`. I want `<the right structure>` — walk me through that instead." | "Its plan `<misses X>`. That's the moment to correct it, not after it's coded — I'm redirecting now." |
| **2** | **Implement to spec** (one function at a time) | Your prompt encodes taste, not just intent | "Good — `<confirm plan in one line>`. Implement **just `<the first function/helper>`** in `<language>`. Constraints: `<senior detail, e.g. sentinels / injectable clock / mark-on-enqueue>`. **Docstring with time+space complexity, meaningful names (no single letters except loop indices), no external libraries.**" | "I'm having it build one piece at a time so I can review each before the next — not generate the whole thing and hope." |
| **3** | **Interrogate (find the bug)** | You review by reading, not by crashing | "Before I trust this, trace `<the risky operation>` on `<a small concrete input>` step by step. Specifically `<point at the suspect line>` — is `<the exact value/pointer/index>` correct here? Where could this break?" | "Reading this before I run it. `<Point at the line.>` Let me trace `<input>` through it — I want to see the pointers/keys/indices myself." |
| **4** | **Verify empirically** | You make correctness **observable** | "Write and run tests covering: `<normal>`, `<edge: empty / capacity-1 / single>`, `<the case that would fail under the bug we just fixed>`. **State what these tests do and don't prove.**" | "Running it now, not just eyeballing. This test specifically targets the `<bug>` we just fixed. Note: these prove correctness, not performance / the concurrency path)." |
| **loop** | **Loop 2->3->4 per component** | You never batch untested code | *(repeat Beats 2-4 for each next function)* | "That piece is verified — moving to `<next function>`." |
| **5** | **Extend / scale** | You go past the base problem into real trade-offs | "Confirm every operation is still `O(...)` and explain why. Then adapt to `<harder variant: distributed / streaming / dynamic / concurrent>`. What changes, where's the hard part, and the failure modes at scale?" | "Base case works. Let me push on scale — `<the variant>` — because that's where the real design decisions are." |
| **6** | **Explain back (agent aside)** | You **own** it — the deliverable they actually want | *(agent aside)* | "The invariant is `<one sentence>`. It composes `<structures>`; complexity is `<...>` because `<...>` . The one line that's easy to get wrong is `<the bug>`. I chose `<this>` over `<alternative>` because `<reason>`." |



---

## The three rules to bake into EVERY prompt (Beats 1, 2, 5)

These are the standing constraints that make the agent's output reviewable and senior-looking. Say them once in Beat 2 and they carry; restate briefly if the agent drifts.

### 1. Time + space complexity — always stated, always in the code
*   **Beat 1: demand complexity *in the plan*,** in terms of the real variables (N, K, m-n, L...), **before** any code. A plan without a complexity claim is incomplete.
*   **Beat 2:** Every function's **docstring states its time and space complexity**. Writing it down makes a wrong claim visible on the page (docstring says O(1), body has a loop -> contradiction you catch instantly).
*   **Beat 5:** Make the agent **re-confirm** complexity after edits, and justify it ("why is this still O(1)", "why doubly not singly linked", "why O(N log K) not O(N·K)").
*   **Prompt fragment: "State time and space complexity in each docstring, in terms of `<vars>`, and flag any operation that isn't the complexity we agreed."**

### 2. Docstrings — contracts, not decoration
*   Every function gets a short docstring covering: **what it does, its complexity, and any side effect** (mutation, ordering, recency-refresh — the side effects are where bugs hide).
*   Keep them terse. A docstring is a contract you can check the body against, not prose.
*   **Prompt fragment: "Each function has a short docstring: one line on behavior, the time/space complexity, and any side effect (mutation, ordering)."**

### 3. Meaningful variable names — intention-revealing
*   **No single-letter names except conventional loop indices** (`i`, `r`, `c`, `dr`, `dc` for grid deltas are fine).
*   Names should reveal role: `refill_rate` not `rr`; `last_refill` not `t0`; `descended_node`/`nxt` not `n2`; `frontier` not `q2`.
*   **Why this earns points: the LRU bug (`_add_front` without `_remove`) and the Word-Search bug (recursing from `node` vs `nxt`) are both easier to catch when the two things have distinct, meaningful names.** Good naming is bug-prevention, and the interviewers know it.
*   **Prompt fragment: "Use intention-revealing names — no single letters except loop indices; a reader should infer each variable's role from its name."**

---

## Continuous verification — loop Beats 2->3->4, never batch (interviewer red flag #1)

The single most-cited failure in real interviews: **"not running code after each generation, then hitting cascading failures at the end."** The defense is structural — **do not treat Beat 2 as "generate the whole solution."** Build and verify one component at a time.

```
Beat 0 — scope (humans)
Beat 1 — plan (+ 1.5 correct if it drifts)
    ┌ for each function / helper:
    │  Beat 2  build THIS piece
    │  Beat 3  read + trace THIS piece      <- loop
    │  Beat 4  run a quick test on it
    └
Beat 5 — extend / scale (once all pieces verified)
Beat 6 — explain back (humans)
```


*   **Ask for one function at a time** in Beat 2 ("implement just `<X>`"), not the whole class.
*   **Run something after every meaningful change** — even a one-line sanity check beats a 200-line dump you test at the end.
*   **If a test fails, stop and diagnose out loud** before asking for a fix — "the recency isn't refreshing; let me see why," then direct the precise fix.
*   **Why it scores:** it kills the article's "core illusion" — AI *feels* productive while you're actually accumulating untested, possibly-broken code. Incremental verification is the antidote and the visible signal.

## Communication is a graded output, not background noise (interviewer red flag #2)

Interviewers grade communication as one of four axes, and the named anti-patterns are precise: **"going quiet for long stretches while prompting"** and **"narrating after the fact ('I just asked') instead of before ('I'm going to ask')."** Real example: Canva interviewers **pause and ask "what does this code do?"** — they will actively test whether you understand the agent's output.

Rules:
*   **Speak before every prompt and every transition** — the **Say out loud** column above is the script. Future tense: **"I'm going to ask it to..."**, never past tense.
*   **Read the agent's output aloud** as you review it (that *is* Beat 3). Silent reading looks like blind acceptance.
*   **Announce agreement/disagreement** when the agent proposes something — **"I agree with the sentinel idea"** or **"I don't want that, because..."**. Never silently accept or silently pivot.
*   **Be ready for the interrupt.** If they ask "what does this code do?", your Beat-3 read-aloud habit means you're already doing it — answer with the invariant, not a line-by-line recital.
*   **No silent stretches.** If you're thinking, say you're thinking: "Give me a second — I'm checking whether the eviction pointer is right."

## Tempo rules (the meta-signal)

*   **One beat at a time.** Never let the agent emit a full solved solution in one shot. The review pauses are the interview.
*   **Narrate intent before each prompt** — say *why* you're asking, out loud, to the interviewers. The judgment in the pauses is what they're scoring.
*   **Verbalize any bug you spot before the agent does.** That is the single strongest signal you can send.
*   **Direct fixes precisely** — "call `_remove` then `_add_front`", not "fix it".
*   **Name when you'd NOT use the agent** — tiny changes, or when you need to reason through an invariant yourself.

## What they're scoring (the 4 axes interviewers actually grade)

| Axis (from real interviewer feedback) | How you show it |
| :--- | :--- |
| **Problem-solving & approach** | Beat 0–1 — you plan before opening the chat, seed a spec not the raw problem |
| **Control over the AI** | Beat 0 (you own the DS choice) + Beat 1.5 (redirect on drift) + precise fixes, not "fix it" |
| **Verification habits** | Beats 2->3->4 looped per function — run continuously, read line-by-line, never batch |
| **Communication** | Spoken line *before* every prompt/transition; read output aloud; ready for "what does this do?" |
| **(bonus) Depth** | Beat 5 — scale/variant trade-offs |
| **(bonus) Ownership** | Beat 6 — you explain the invariant, not the lines |



**The core illusion to avoid: AI makes you feel productive even when you're in trouble.** The looped verification and constant narration are what keep you (and the interviewer) grounded in what's actually working.

## Quick-fire prompt strip (tear-off for live use)

| Beat | Say this |
| :--- | :--- |
| **0** | *(to humans)* "Restate + constraints + my approach vs. the alternative; agent will plan independently to check me." |
| **1** | "Plan only — structures, algorithm, **time/space complexity in `<vars>`**, failure mode. No code yet." |
| **1.5** | *(if drift)* "That plan `<misses X>` — do `<right structure>` instead." |
| **2** | "Implement **just `<one function>`**. **Docstrings w/ complexity, meaningful names, no libs.** `<senior detail>`." |
| **3** | *(aloud)* "Reading before running. Trace `<op>` on `<input>` — is `<value>` right on `<line>`? Where does it break?" |
| **4** | "Test `<normal / edge / the-bug-case / scale>`. Say what the tests prove." |
| **↺** | *(aloud)* "Verified — next function." **(loop 2->3->4, don't batch)** |
| **5** | "Confirm complexity. Now do `<harder variant>` — hard part + failure modes?" |
| **6** | *(to humans)* "Invariant is `<...>`; complexity `<...>`; fragile line is `<...>`; chose `<X>` over `<Y>` because `<...>`." |

