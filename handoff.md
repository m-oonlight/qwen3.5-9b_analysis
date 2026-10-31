# 250q exam: handoff

Last updated: 2026-10-31

## What this is

This is the hard_v3 exam: 250 questions for studying how GGUF quantization (Q8_0, Q6_K, Q5_K_M, Q4_K_M, Q3_K_M, Q2_K) changes the reasoning of Qwen3.6-35B-A3B. The study plan is in `../paperinfo.txt`; the run settings are in `../systemconfig.txt`.

This folder is meant to be self-contained. The three files below are all that's needed; the older generator and checker pipeline in `../hard_v3_handoff/` is no longer part of the workflow.

| File | Contents | Show to the model? |
|---|---|---|
| `exam.json` | 250 questions, a system prompt and usage notes | Only the question text (see open item 2) |
| `answer_key.json` | Exact expected final JSON for the 200 math/logic questions | **Never** |
| `hidden_tests.json` | 2,451 test cases for the 50 coding questions | **Never** |

There are 50 families of 5 questions each. The 200 math/logic questions span arithmetic and number theory, algebra, combinatorics, exact probability, optimization and graphs, and sequences and recurrences, with 25 questions each; logic and data constraints have 50. The remaining 50 are Python coding tasks.

## Audit (2026-10-31)

Every question was solved independently from its text, by brute force with exact fractions for math and by reference implementations for coding, and the results were compared with the key and the tests.

- **Answer key:** all 200 entries are correct.
- **Hidden tests:** all 2,451 expected outputs are correct, and every input obeys the constraints stated in its question. Reference solutions finish in well under the 10 s limit.
- **Fairness:** every question is solvable and objective, with exactly one correct answer under a careful reading.
- **Kept on purpose:** about 15 questions are correct but take a lot of hand bookkeeping without tools (for example PERIODIC_MATRIX-04/05, COUPLED_RECURRENCE-03/04, RESTRICTED_PERMUTATION-01/05, PERMUTATION_PROBABILITY, MULTISET_WORDS-03). They were kept on purpose: long enumeration and arithmetic are where quantization failures are expected to show.

## Changes made to `exam.json` (2026-10-31)

Only the wording changed. The `Data`, `Question parameters` and answer-schema text is byte-identical to the original in all 200 math questions. Coding-question inputs and outputs are unchanged. `answer_key.json` and `hidden_tests.json` were not modified. The original `exam.json` is preserved in `../250q.zip`.

- Removed boilerplate that didn't apply, such as prefix and lexicographic rules where nothing is compared, "ordered vectors/strings" in plain counts, and job/tree/assignment wording in unrelated optimization questions. The average question went from about 1,770 to 1,430 characters.
- Repeated-trial probability questions now state plainly: "A trial is one independent run of the whole experiment above, so each trial succeeds with probability exactly R." The old wording about "retaining conditioning" was ambiguous.
- Clarified the following:
  - integral orientation (INTERPOLATION-05);
  - "labels a and b" (PRECEDENCE_ORDERS);
  - integer_system score wording, so that x no longer means both the triple and its first coordinate;
  - "comparing vertex labels as integers" (DEPENDENCY_DAG-01);
  - pattern/string roles (STRING_DP-03);
  - a variable `x` that some score rules used without defining it.
- The coding rules are now a bullet list that matches what the original grader enforces. Internal jargon was removed, and the 10 s CPU / 20 s wall limits are stated.
- A second independent review compared all 250 old and new questions and recomputed the affected answers. It found no change in meaning or answers.

## Open items before the full Q8 to Q2 run

1. **Freeze the run settings.** Q4 showed about 112 MiB of process swap under the shared 30-CPU-expert-layer placement, which fails the zero-swap rule. Q6 and Q5 still need memory checks. See `../systemconfig.txt`.
2. **System prompt.** `exam.json` (`how_to_use` / `system_prompt`) says to send a system prompt, but `../systemconfig.txt` says "No custom system message". Pick one, apply it identically to every quantization, and update `how_to_use` to match.
3. **Scoring.**
   - Grade math answers by exact comparison of the final JSON against `answer_key.json`, using a plain script rather than an LLM.
   - Grade coding answers by running them against `hidden_tests.json` in an isolated sandbox, because the code is untrusted.
   - Use an LLM only afterwards, to label failure types. It must never decide scores.
4. **Recommended Q8 pilot.** Run 20 to 30 questions with one seed to check that Q8 accuracy is mid-range (not at a floor or ceiling) and that answers fit in the 16,384-token thinking budget.
5. **Record truncation separately.** A response cut off by the token limit should be reported apart from wrong answers, so a lack of budget isn't mistaken for lost reasoning.

## Analysis notes

- Families have only 5 questions × 3 trials, so family-level differences are noisy. Rely more on category-level and overall comparisons.
- Questions within a family share a setup and aren't independent. Treat each family as a cluster in the statistics.
- Three seeded trials (73191, 73192, 73193) are needed because sampling at temperature 1.0 is random. They measure consistency, not just accuracy.
