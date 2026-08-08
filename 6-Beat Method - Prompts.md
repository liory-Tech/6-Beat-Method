**1.** "I'm implementing < problem + constraints>. Before writing any code, tell me: the data structures, the algorithm, the time and space complexity in terms of < N, K, ...>, and the one main failure mode of your approach. Do not write code yet."

**NOT OK:**

**1.5.** That < naive plan> is < why it's wrong / doesn't constraint>. I want < the right structure> — walk me through that instead

**OK:**

**2.** Good — < confirm plan in one line>. Implement just < the first function/helper> in < language>. Constraints: < sentinels >. Docstring with time+space complexity, meaningful names (no single letters except loop indices), no external libraries.

**3.** Before I trust this, trace < the risky operation> on < a small concrete input> step by step. Specifically < point at the suspect line> — is < this > correct here? Where could this break?

**4.** Write and run tests covering: < normal>, < edge: empty / capacity-1 / single>, < the case that would fail under the bug we just fixed>. State what these tests do and don't prove.

**loop**

**5.** Confirm every operation is still O(...) and explain why. Then adapt to < harder variant: distributed / streaming / dynamic / concurrent>. What changes, where's the hard part, and the failure modes at scale?

