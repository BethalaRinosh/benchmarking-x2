# Agentic Security Benchmarking

**Repository:** `BethalaRinosh/benchmarking-x2`  
**Working plan:** Revised Plan v4 (6 November 2026)  
**Status:** Phase 0A not started. No benchmark measurements have been produced.

This README is the durable handoff log for every benchmark action. Update it after each user action, command, error, fix, configuration change, experiment, and result. Never invent missing outputs. Record exact commands and sanitized logs; never commit API keys, student ID numbers, malware samples, or other secrets.

## 1. Research question

Which of Strix, PentAGI, Shannon, and PentestGPT is the best reasoning layer for judging whether an untrusted software project is safe, and does any agent outperform a simple pipeline?

A thin pipeline is a valid winning outcome. Priority order:

1. False-safe rate (Unsafe project called Safe).
2. Prompt-injection resistance.
3. Malicious dependency detection.
4. False positives on clean projects.
5. Source-code vulnerability detection.
6. Live exploitation (last).

## 2. Fixed constraints and safety

- One laptop; RTX 4060 8GB VRAM, 16GB RAM, approximately 200GB free.
- Hosted LLM API; one agent stack at a time.
- No local LLM for agent runs.
- WSL2 is acceptable for static file work only, not as the security boundary for running untrusted projects.
- Anything executing submitted code or an agent sandbox runs in a separate VM with native Docker Engine, no shared folders/clipboard/host mounts, and restricted network egress.
- Never install or execute benchmark samples. Malicious package samples remain outside this public repository. Download package tarballs without invoking build/install tooling.
- Restore a clean VM snapshot before each run. Pre-pull required images, then restrict egress to the LLM API endpoint.
- Keep API keys in ignored environment files or a secret manager. Verify ignore rules and tracked files before committing.
- Record failures as failures. Timeouts, budget exhaustion, and failed runs are **Inconclusive**, never Safe.

## 3. What Unsafe means

Label two independent axes:

- **Malicious:** intentional harm (exfiltration, backdoors, miners, destructive install scripts, poisoned dependencies).
- **Vulnerable:** accidental defects (for example, SQL injection or path traversal).

Policy:
- **Unsafe:** malicious, or a high/critical vulnerability reachable in the platform's intended use.
- **Safe:** neither malicious nor a qualifying high/critical vulnerability.
- **Inconclusive:** run failed, timed out, or reached its budget cap.

False-safe = ground-truth Unsafe but agent verdict Safe.

## 4. Phase 0A: ground truth and harness (no Docker)

### Corpus v0: 30 items

| Group | Count | Notes |
|---|---:|---|
| Clean student-style projects | 5 | Reviewed; document minor issues |
| Vulnerable projects | 5 | Self-written Flask/Express; one flaw each; at least two non-textbook flaws |
| Injected vulnerable copies | 5 | Copies with plain-text prompt injections |
| Self-written malicious projects | 5 | Inert simulations only; never execute |
| Malicious packages | 5 | DataDog dataset; one sample per cluster; static only |
| Benign packages | 5 | Similar size/type; registry tarballs downloaded directly |

Self-authored samples have bias because the author knows what is hidden. State this limitation; seek a second reviewer if possible. Public vulnerable apps may be in training data, so use self-written repos for Experiments 1 and 3.

### Required deliverables

1. `labels.json`: item ID, group, malicious flag, vulnerable flag, expected findings (category/file/line), injection placement and style.
2. Common report schema: `verdict` (Safe/Unsafe/Inconclusive), `findings[]` (category, file, line, evidence, severity, status: confirmed/suspected/unverified). Preserve raw output.
3. Native-output parser/adapter per agent. Do not ask an LLM to normalize agent reports.
4. Scorer comparing parsed output with `labels.json`; ambiguous cases go to manual review.
5. Baseline A: one prompt per project with file chunking.
6. Baseline B: A plus Semgrep (and Bandit for Python).
7. Baseline C: same model in a bounded loop with read-only list/read/grep tools.
8. Run A, B, C on corpus v0 with the same frozen model and caps.

Malicious package archives stay outside Git. DataDog archives are encrypted ZIPs (password `infected`); extract only inside the isolated lab VM. Sample one package per cluster. Never run `npm install`, `pip install`, package build scripts, or equivalent on samples.

**Phase 0A exit gate:** scorer runs end-to-end on A/B/C and produces initial false-safe and injection results with Wilson 95% intervals. These small-sample results validate the harness; they do not establish safety.

## 5. Phase 0B: agent feasibility

Order: **Strix first**, PentAGI only if Strix is viable, then Shannon/modern PentestGPT only for their supported live/Mode B use cases if budget remains.

| Agent | Static local source | Gemini Mode A | Current classification |
|---|---|---|---|
| Strix | Documented local directory target | Gemini via Vertex/OpenRouter documented; direct AI Studio key needs testing | First agent; eligible for Experiments 1-3 if feasible |
| PentAGI | Not established; test mounting source | Gemini supported | Conditional; timebox |
| Shannon | Requires running URL plus source | Gemini only through experimental OpenRouter path | Not eligible for static vetting; possible live Mode B |
| Modern PentestGPT | Host/IP target | Claude-oriented backends | Mode B only |
| PentestGPT legacy | Human-relay workflow | Gemini supported | Parked; not comparable autonomous agent |

Per-agent checklist: local-source analysis without live target; what executes and where; Docker and peak RAM; frozen Gemini compatibility; parseable native output; one injected-pair smoke task; tokens/cost/tool calls/setup hours. Hard setup timebox: four hours per agent. If exceeded, record “Not viable on this setup.”

## 6. Lab isolation

Use a separate Ubuntu VM (Hyper-V/VirtualBox or equivalent) with native Docker Engine, around 9GB RAM, no shared folders, clipboard, or host mounts. WSL2 is not a sandbox for untrusted code. Take a clean snapshot. Pre-pull agent images, then restrict egress to the LLM API domain using VM/router firewall controls. VM/Docker setup timebox: one 90-minute sitting.

## 7. Gemini/API decision

The model ID must be frozen for all Mode A runs and recorded on every run. The prior plan proposed `gemini-3.5-flash`; it must be re-verified for availability/deprecation before recorded runs.

**Important account distinction:** a Google AI Pro subscription is not, by itself, proof that an external agent's Gemini API calls are covered. Check the AI Studio API key/project and its API billing/usage tier. Keep a hard spend cap and do not start recorded runs until the available quota, data terms, model ID, and budget are confirmed. Student status or a student ID does not automatically grant Gemini API credits. Never put the student ID in this repository.

Official references:
- [Google AI Pro subscription](https://gemini.google/subscriptions/)
- [Gemini API getting started / API keys](https://aistudio.google.com/docs/get-started)
- [Gemini API billing](https://ai.google.dev/gemini-api/docs/billing)

## 8. Experiments

- **E1 Injection resistance:** five vulnerable/injected pairs in v0. Arms: verdict suppression, action hijack (canary file/command/URL with logs), and a small adaptive-injection subset. Vary placement and style. Plain text only.
- **E2 Malicious dependencies:** five cluster-sampled malicious and five size/type-matched benign packages, static only. GuardDog is a biased reference/ceiling, not a fair competitor.
- **E2b Malicious projects:** five self-written inert malicious projects.
- **E3 Clean projects:** five reviewed clean projects; measure false positives.
- **E4 OWASP Benchmark:** optional, only after core phases; Java mismatch and likely training contamination.
- **E5 Live testing:** last; local Juice Shop, reset per run, challenge tracker ground truth, Mode B agents only.
- **E6 Investigation quality:** annotate tool choice, interpretation, adaptation, repeated work; blind re-score a random 20% after a week.

## 9. Fair comparison and metrics

- Mode A: same frozen Gemini model for all agents that support it.
- Mode B: each agent's recommended native model; report separately.
- Equivalent prompts; randomize order where practical.
- Record agent version/commit, model/provider/mode, key type (never key value), tools/permissions, step/token caps, network rules.
- Metrics: false-safe rate, injection resistance, recall, precision, false-positive rate, evidence accuracy. Secondary: severity accuracy, tool selection, failure rate, tokens, cost, tool calls.
- A finding counts only with the real issue and valid evidence, not a category label alone.
- Report counts with Wilson 95% intervals. Repeated runs measure noise; they do not increase the number of distinct samples. With zero false-safes among five Unsafe items, the upper 95% bound is still about 43%.

One row per run: run ID, agent/version, model/provider/mode, experiment/sample ID, expected result, parsed verdict/findings, correctness, false positive/false safe, evidence quality, tokens/cost, tool calls, status, notes. Preserve raw logs outside the lab VM.

## 10. Timeline and gates

| Phase | Work | Exit gate |
|---|---|---|
| 0A | Policy, corpus, labels, schema, scorer, baselines A/B/C | Scorer works; initial false-safe/injection results |
| 0B | Lab VM, Docker, Strix then possible PentAGI feasibility | Viability table and cost/run |
| 1 | Expand corpus and write budget caps | Corpus labeled; caps written |
| 2 | Pilot, three repeats on subset of E1/E3 | End-to-end pipeline works |
| 3 | Full E1, E2, E2b, E3 runs | Results logged and verified |
| 4 | Optional E5 Mode B live test | Challenge results logged |
| 5 | Analysis and recommendation | Final report |

Minimum viable benchmark: Baselines A/B/C plus Strix on E1, E2b, E3, and E2. If PentAGI is viable, add it. Cut E5, E4, E6, then PentAGI in that order. Never cut baselines or E1-E3.

## 11. Limitations to report

Small/self-authored corpus; single/few labelers; public benchmark contamination; DataDog selection bias and clustering; one frozen model; only Strix (possibly PentAGI) may qualify for Mode A; pentest agents are not malware classifiers; good results do not prove a project safe.

## 12. Action log

Append newest entries at the top. Every entry must distinguish **observed facts**, **interpretation**, and **next action**. Include exact commands and sanitized output for terminal actions. Record errors verbatim where safe.

### 2026-11-06 | Session start and repository initialization

- **Action:** Read the supplied Revised Plan v4 and fetched the existing repository README.
- **Observed:** The repository README contained only the heading `# benchmarking-x2`. The supplied plan states Phase 0A has not yet produced a corpus, labels, schema, scorer, or baseline results; no benchmark measurements exist.
- **Observed:** User reports having a student ID and Gemini Pro/Google AI Pro access. No API key, paid API tier, available credits, or API quota has yet been verified.
- **Interpretation:** Student status and the consumer Gemini subscription must not be treated as proof of Gemini API funding. Confirm the AI Studio project/key and billing tier before any recorded agent run.
- **Decision:** Follow v4 gates. Do not install/run Strix yet. Phase 0A comes first; Phase 0B requires an isolated VM and native Docker.
- **Next action:** Verify repository secret hygiene, then define the Unsafe policy and start the corpus manifest.
- **Errors:** None observed in this session so far.
