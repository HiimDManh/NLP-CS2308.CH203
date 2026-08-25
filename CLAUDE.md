# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

This is a course deliverable repo for **CS2308.CH203 — Chuyên đề nghiên cứu và ứng dụng về Xử lý ngôn ngữ tự nhiên**. The whole repo revolves around one paper: **Self-RAG: Learning to Retrieve, Generate, and Critique through Self-Reflection** (Asai et al., ICLR 2024, arXiv:2310.11511, local copy at `paper/2310.11511v1.pdf`).

The work is split into two tracks defined in `SELF_RAG_SEMINAR_PROJECT_GUIDELINE.md` (the master planning doc):

1. **Seminar track** — present the paper (motivation, reflection tokens, training/inference pipeline, results, limitations).
2. **Final project track** — build a small **"Self-RAG-inspired" prototype**: a baseline RAG vs. a self-reflective RAG (retrieval-decision → relevance judge → generate → support judge → usefulness judge), compared on a QA dataset, without needing to fine-tune a model like the original paper (reflection signals are simulated via prompting/scoring rules instead).

**Current state: final-project implementation is underway** under `final/` (see `final/PIPELINE.md` for the concrete architecture/roadmap and its checklist for what's actually built vs. still planned). Most content documents are written in **Vietnamese**; keep diacritics correct when editing them.

## Repository map

- `SELF_RAG_SEMINAR_PROJECT_GUIDELINE.md` — the source of truth for both tracks: seminar slide outline, Self-RAG concept summary, and the full final-project design (module list §25, baseline vs. Self-RAG-inspired architecture §24, evaluation plan §27, weekly plan §29, team split §30).
- `seminar/SELF_RAG_SEMINAR_DETAILED_GUIDE.md` — slide-by-slide script (~26 slides) for the seminar, each slide annotated with which paper section/table it must cite (e.g. Table 1 for reflection tokens, Algorithm 1 for inference), plus a prepared Q&A bank.
- `seminar/NOTEBOOKLM_SLIDE_PROMPT.md` — a ready-to-paste NotebookLM prompt that regenerates the slide deck from the paper PDF + the detailed guide above.
- `paper/2310.11511v1.pdf` — the original Self-RAG paper (primary source of truth for any factual claim used in slides/report).
- `slide/The_SELF-RAG_Blueprint.pptx` — the slide deck.
- `final/dataset/VLQA/` — the real **VLSP 2025 DRiLL** shared-task dataset (Deep Retrieval in the expansive Legal Landscape, https://vlsp.org.vn/vlsp2025/eval/drill), not a toy corpus:
  - `legal_corpus.json` / `legal_dataset.json` — same 2157 Vietnamese legal documents in two forms: flat (`law_id` + `content[]` articles keyed by `aid`) vs. hierarchical (`href`, `title`, chapter → section → article). **`aid` is globally unique across the whole corpus (0–59635, 59636 articles total, verified no duplicates)** — build a single global `aid → text` lookup, not a per-document one.
  - `train.json` — **2190 labeled examples**: `qid`, `question` (real colloquial "hỏi luật sư" style user questions), `relevant_laws` (gold list of `aid` — avg 1.34, max 9, min 1 per question, never empty), `answer` (human-written free-form legal-advice answer, not an extractive span). This is the real eval/dev resource — no need to hand-author a question set.
  - `public_test.json` (312) / `private_test.json` (627) — same schema but `relevant_laws: []` and `answer: ""` (labels withheld for the official shared-task leaderboard). **Not usable for local evaluation**; only meaningful if actually submitting predictions to the VLSP2025 DRiLL leaderboard.
  - `readme.md` — the official schema description for the above files.
  - **The `*.json` files here are gitignored, not committed** — `legal_corpus.json` (~117MB) and `legal_dataset.json` (~134MB) exceed GitHub's 100MB hard push limit, so by decision the whole dataset is kept out of git rather than using Git LFS. They only exist on local disk / Google Drive; a fresh `git clone` will *not* bring them — see `final/PIPELINE.md` §6.1 for the one-time manual Drive upload step. If these files appear missing after a clone, that's expected, not a bug.
- `final/PIPELINE.md` — the concrete build plan for the final project on top of the real dataset: 3-system architecture (No-RAG / Standard RAG / Self-RAG-inspired), the `final/notebooks/` → `final/artifacts/` data flow, step-by-step Colab operating instructions (Drive mount, `git clone` from `origin` at `github.com/HiimDManh/NLP-CS2308.CH203`, notebook run order, troubleshooting), and what the finished deliverable looks like. Check the checklist at its end for current progress before assuming a notebook exists.
- `final/notebooks/01_retrieval_baseline.ipynb` — Week-1 deliverable: chunk `legal_corpus.json` (paragraph-first splitter, 800-char/150-overlap fallback), embed with `bkai-foundation-models/vietnamese-bi-encoder`, build a FAISS `IndexFlatIP`, and evaluate Recall@k/Precision@k/MRR against a fixed 250-question dev split carved out of `train.json` (seed 42, qids persisted to `final/artifacts/dev_split_qids.json` so later notebooks reuse the same split). Designed to run in Colab **opened directly from GitHub** (never `git clone` into a mounted Drive — Drive's FUSE layer breaks git's pack-object writes, confirmed by a real `invalid index-pack output` failure; see `final/PIPELINE.md` §6.1/§6.4) or locally. On Colab it mounts Drive only as a plain data folder (`DRIVE_DATA_ROOT`, default `/content/drive/MyDrive/NLP-CS2308.CH203-data`) holding `VLQA/` (dataset, uploaded manually once) and `artifacts/`; locally it derives both from `REPO_DIR`. Persists the FAISS index/embeddings/metrics under `artifacts/` (git-ignored — regenerate by rerunning, don't hand-edit).
- `final/notebooks/02_generator_baseline.ipynb` — Week-2 deliverable, the "Standard RAG" system (fixed top-5 retrieve → generate, no reflection). Loads the artifacts from notebook 01 (fails fast with a clear error if they're missing), re-defines `retrieve()` locally (notebooks are independent, no cross-notebook imports), and calls **Groq** (`groq` SDK, chat-completions style, model `llama-3.3-70b-versatile`, key read from Colab Secrets as `GROQ_API_KEY` or env var locally). **Originally used Google Gemini** (`google-genai`, `gemini-2.5-flash`) but switched after the user's Gemini API key started requiring Google Cloud billing setup — Groq's free tier needs no billing. If Gemini free-tier access ever becomes viable again, swapping back only touches cell-2 (pip install), cell-3 (key var name), cell-9 (client init), and the API call inside `generate_answer()` in cell-11. Generation is checkpointed to `final/artifacts/standard_rag_results.jsonl` one line at a time (resume-safe across Colab disconnects — reruns skip already-done `qid`s). Notebook 03 will produce the analogous `self_rag_results.jsonl` for the Self-RAG-inspired system.
- `research/ecc2-codebase-analysis.md` — **unrelated leftover file**: a code-quality report about a different, unrelated Rust project (`ecc-tui`). It has nothing to do with Self-RAG or this course project; ignore it unless the user asks about it specifically.
- `.codex/`, `.codex-plugin/` — Codex CLI configuration (MCP servers, agent roles, sync scripts). Not used by Claude Code; only relevant if the user is switching tools.

## Self-RAG concepts worth keeping straight across sessions

Reflection tokens (the paper's core mechanism, Section 3 / Table 1):

| Token | Question it answers | Labels |
|---|---|---|
| `Retrieve` | Is retrieval needed at all? | Yes / No / Continue |
| `ISREL` | Is this passage relevant? | Relevant / Irrelevant |
| `ISSUP` | Is the answer supported by evidence? | Fully / Partially / No support |
| `ISUSE` | Is the answer actually useful? | Useful / Partially / Not useful |

Standard RAG: `Question → Retrieve top-k → Generate`. Self-RAG: `Question → decide Retrieve? → (retrieve) → judge ISREL per passage → Generate → judge ISSUP → judge ISUSE → final answer`. The recurring three-word summary used throughout the docs is **Adaptive Retrieval, Self-Critique, Controllable Generation**.

## Working conventions

- There's no build/lint/test tooling to run — treat this as a writing/research repo, not a software codebase.
- When asked to extend the seminar or the final-project plan, check `SELF_RAG_SEMINAR_PROJECT_GUIDELINE.md` first for scope/architecture decisions already made, and `seminar/SELF_RAG_SEMINAR_DETAILED_GUIDE.md` for the citation-to-paper-section mapping style already established (every claim in the seminar guide cites a specific paper section/table — keep that pattern for new slide content).
- If/when the final project's code actually gets implemented, it should follow the architecture already spec'd in guideline §24–§25 (document ingestion → embedding/vector store → retriever → baseline generator → retrieval-decision module → relevance/support/usefulness judges → response formatter) rather than inventing a new structure.

## `.agents/` skill library

`.agents/*/SKILL.md` is a generic skill library written for **Codex CLI** (see `.codex/AGENTS.md`, which documents them as auto-loaded there), *not* for Claude Code — none of these appear in Claude Code's own Skill-tool listing, so they won't be invoked automatically. To use one, read the relevant `.agents/<name>/SKILL.md` directly and follow its instructions manually. Skills actually relevant to this repo's work:

- `article-writing` — long-form writing in a consistent voice; useful for the seminar report/paper.
- `deep-research` — multi-source cited research; useful for backing up claims beyond the Self-RAG paper itself.
- `agentic-engineering`, `agent-harness-construction`, `design-system` — general engineering/design skills, only relevant if the final project grows into real code.

The remaining skills (`healthcare-cdss-patterns`, `healthcare-emr-patterns`, `healthcare-eval-harness`, `healthcare-phi-compliance`) are boilerplate imported from an unrelated healthcare project and don't apply here.
