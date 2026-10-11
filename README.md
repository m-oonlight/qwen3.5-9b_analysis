# Qwen3.5-9B Quantization Analysis

This repository preserves the setup, test suite, scoring tools, and progress of a study comparing **Q8_0, Q6_K, Q5_K_M, Q4_K_M, and Q3_K_M** quantizations of Qwen3.5-9B.

The study measures task accuracy, completion reliability, token usage, prompt-processing and generation speed, elapsed time, and GPU memory. It also examines whether particular task categories become less reliable as compression increases. These are measures of performance on this suite, not a universal intelligence score.

## Read these files first

| File | What it contains | When to use it |
|---|---|---|
| [`technical_handoff`](technical_handoff) | Experiment settings, hardware/software versions, previous failures and fixes, snapshot progress, and reproduction instructions | Start here to understand or continue the project |
| [`exam.txt`](exam.txt) | All 60 questions without answers | Inspect the exam or supply individual questions to the benchmark model |
| [`examkey`](examkey) | The same questions with 50 exact answers and 10 coding reference solutions | Validate answers or ask an independent reviewer to audit the exam |
| [`Benchmark-handoff.zip`](Benchmark-handoff.zip) | The runnable benchmark package, objective scoring data, audit evidence, and saved results | Download and extract this to reproduce the experiment |

`technical_handoff` and `examkey` are plain-text files, despite having no filename extension.

**Do not give `examkey` or the private scoring data to the model being benchmarked.** Each question is sent separately in a fresh request, rather than sending the entire exam in one conversation.

## Current experiment settings

| Setting | Value |
|---|---|
| Context capacity | 32,768 tokens |
| Output cap | 12,288 tokens, including reasoning and final answer |
| Reasoning | Enabled, with a fixed 2,048-token reasoning budget |
| Temperature | 1.0 |
| Top-p / top-k / min-p | 0.95 / 20 / 0.0 |
| Presence penalty | 1.5 |
| Trials per quantization | Three, using seeds 73191, 73192, and 73193 |
| KV cache precision | f16 for both keys and values |
| Batch / microbatch | 2,048 / 512 |
| Parallel sequences | One |
| CPU generation / prompt threads | 16 / 16 on the original machine; explicitly preserved in the exported launcher |
| Flash Attention | Auto |
| GPU placement | All model layers and input embeddings on CUDA; KV cache and recurrent state on CUDA; automatic fitting disabled |
| Request timeout | 1,800 seconds |

The same settings and task/seed pairings must be used across quantizations. A fixed context capacity does not mean every prompt has the same length. Speed comparisons should match tasks and workloads.

The original machine uses an **RTX 4060 Ti 16GB**, a **Ryzen 9 7950X**, and **32GB RAM**. Exact runtime commit, model repository revision, model hashes, Docker image digest, and additional runtime defaults are recorded in the handoff package.

## What the exam measures

The audited v2.1 exam has:

- 20 mathematical and logical reasoning questions.
- 20 data reasoning questions.
- 10 instruction-following questions.
- 10 coding questions.

The 50 non-coding answers were independently recomputed. Coding solutions are evaluated against **4,076 hidden cases** in isolated Docker containers. Reference solutions passed those cases. Other functionally equivalent solutions can also pass.

Scoring separates substantive correctness from schema or formatting failures, truncation, and infrastructure errors. Docker belongs to the evaluator; the model does not receive Docker access. Model-generated code is not executed directly on the host.

The exam and answer keys are now publicly released. Do not describe this as a guaranteed contamination-free test. Q8 is itself quantized and serves as the comparison reference, not a full-precision baseline.

## How to read the extracted package

Extract `Benchmark-handoff.zip`. It contains a folder named `Benchmark-handoff`:

| Path inside the folder | Purpose |
|---|---|
| `Technical-handoff.txt` | Complete technical handoff and step-by-step reproduction instructions |
| `config/experiment.json` | Authoritative settings for the exported experiment |
| `datasets/study_v2/public/tasks.json` | Exact machine-readable benchmark prompts |
| `datasets/study_v2/private/answer_keys.json` | Expected answers, reference programs, and hidden coding cases; keep outside benchmark prompts |
| `datasets/study_v2/manifest.json` | Dataset version and integrity hashes |
| `scripts/` | Essential build, download, server, benchmark, scoring, validation, and analysis tools |
| `environment/` | Hardware/build records, runtime properties, model hashes, and sandbox/GPU-placement evidence |
| `raw_results/` | Saved responses, generated reasoning, timings, token counts, and diagnostic or partial study runs |
| `processed_results/` | Earlier diagnostic summaries and reviews |
| `history/` | Obsolete test and scoring versions retained to explain fixes; do not benchmark with these |
| `exam/` | Plain-text reviewer copies with and without answers |
| `snapshot.json` | Export time, partial progress, exclusions, and portability adjustments |
| `FILES.sha256` | Checksums for the exported files |

Older configurations and diagnostic results are historical evidence. They may use different context sizes, output limits, or reasoning settings. Do not combine them with the final study as if they used the same protocol.

## What progress this snapshot contains

The export was created on **October 10, 2026, at 7:06 PM Pacific time**.

At that point:

- All five model files were downloaded and hash verified on the original machine.
- The audited suite and Docker preflight checks passed.
- Four independent calibration questions passed without truncation or infrastructure errors.
- Input embedding weights were moved to CUDA, followed by a successful all-GPU smoke test.
- The full Q8 benchmark was running: **5 of 180 attempts were complete**, with four primary-correct responses.

Those five responses are **partial progress, not a final accuracy result**. The original benchmark can continue after the export, but new results do not automatically appear in this ZIP. Check for a newer export before assuming the stored progress is current.

Earlier failures—including formatting failures, output-limit truncations, repetitive reasoning, and repaired scoring/sandbox issues—are documented in the technical handoff and preserved results.

## Reproduce the benchmark

Read `Technical-handoff.txt` before starting. The main sequence is:

1. Extract the ZIP onto a suitable Linux/NVIDIA machine.
2. Recreate the Python environment and build the pinned llama.cpp commit with CUDA.
3. Copy the verified GGUF weights from the original machine or download them using the pinned revision and hash-verification script.
4. Run the dataset validator and Docker preflight checks.
5. Start one model server with the preserved settings and verify GPU placement from its startup log.
6. Run the benchmark with three trials, then generate its analysis and review page.
7. Repeat for the other quantizations using the same configuration and paired seeds.

**The ZIP does not contain GGUF weights, a compiled llama.cpp runtime, or the original Python virtual environment.** Download/build instructions and identifiers are included. Running background jobs do not transfer with the files.

The runner starts a new run; it does not automatically resume an exported partial run. Keep new runs separate from the saved snapshot. Three trials mean **60 questions × 3 = 180 attempts per quantization**. Several hours may be required; one hour is not guaranteed.

## Continue with another assistant

Give the assistant this repository link and ask it to read `technical_handoff` first, then inspect the extracted package. A useful starting request is:

> Continue this Qwen3.5-9B quantization study. Read the technical handoff, current configuration, dataset manifest, and latest run metadata and summary. Preserve the audited test, scoring rules, model/runtime pins, and inference settings. Distinguish historical diagnostics and partial progress from completed comparison results. Keep private answer keys outside the benchmark model's prompts.
