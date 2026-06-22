<h1 align="center">TriAgent</h1>

<p align="center"><b>An Adaptive Multi-Agent Architecture for Crisis Clinical Decision Support Under Incomplete Information</b></p>

<p align="center">
  <a href="http://creativecommons.org/licenses/by/4.0/"><img src="https://img.shields.io/badge/license-CC--BY--4.0-brightgreen" alt="License: CC BY 4.0"></a>
  <img src="https://img.shields.io/badge/python-3.10%2B-blue" alt="Python 3.10+">
  <img src="https://img.shields.io/badge/built%20with-LangGraph-orange" alt="Built with LangGraph">
</p>

## Overview

**TriAgent** is an agentic, multi-agent **clinical decision support (CDS)** system for emergency / crisis triage under incomplete information. An LLM **orchestrator** runs a ReAct loop over nine specialized tools — selecting and sequencing them per case rather than following a fixed script — while keeping a complete audit trail of every decision.

The system is built on **LangGraph** and evaluated on **real emergency-department data from MIMIC-IV**, using the triage **ESI acuity** as ground truth. Embedded safety verification (RxNav drug–drug interactions, OpenFDA allergy signals), a post-hoc critique agent, and a confidence-gated emergency-escalation pathway are part of the execution graph rather than optional downstream checks.

This repository contains the reference pipeline notebook introduced in our paper:

> **TriAgent: An Adaptive Multi-Agent Architecture for Crisis Clinical Decision Support Under Incomplete Information**
> Ahmed Ibrahim and Ahmed Serag.
> Manuscript under review (2026).

## Key Features

- **Agentic orchestration** — an LLM-driven ReAct orchestrator chooses module sequences per case with no hard-coded routing, while maintaining complete execution audit records.
- **Nine specialized tools** — *assessment* (urgency, clinical retrieval, treatment-plan drafting), *safety* (drug-conflict screening, allergy-risk assessment, safety verdict), and *system* (case-history recall, clinical-note compilation, emergency activation).
- **Agentic RAG** — retrieve → grade → rewrite over 49,000 real MIMIC-IV discharge notes (patient-disjoint from the evaluation set), using `sentence-transformers` embeddings and a FAISS index.
- **Embedded safety** — live RxNav drug–drug interaction screening and OpenFDA allergy signals, a deterministic safety verdict, and a post-loop critique agent that can trigger one remediation pass.
- **Emergency escalation** — a confidence-gated bypass pathway for critical presentations.
- **Auditable** — every run emits a structured audit trace and a natural-language clinical report via `run_trace()`.

## Repository Structure

- **`crisis_cds_agentic.ipynb`** — the complete, self-contained TriAgent pipeline: configuration, intake parsing, FAISS index + agentic RAG, the nine tools, parallel safety execution, the orchestrator ReAct loop, the critique agent, decision memory, LangGraph assembly, and run/report utilities. A small end-to-end demo cell at the bottom (off by default) shows how to run one case.
- **`requirements.txt`** — Python dependencies.

## Installation

```bash
git clone https://github.com/serag-ai/TriAgent.git
cd TriAgent

python -m venv .venv
# Windows
.venv\Scripts\activate
# Linux / macOS
source .venv/bin/activate

pip install -r requirements.txt
```

Create a `.env` file with your API key (the backbone is OpenAI `gpt-5-nano`):

```
OPENAI_API_KEY=sk-...
# optional — enables the HuggingFace medical-critique model
HUGGINGFACE_API_KEY=hf_...
```

## Usage

Open `crisis_cds_agentic.ipynb` and run the cells in order (§1–§12) to load the full pipeline. The per-case entry point is `run_trace()`, which parses a free-text presentation, runs the agent graph, and prints a structured **system audit** plus a **clinical recommendation**:

```python
state = run_trace(
    label="demo",
    case_input=(
        "58-year-old with sudden crushing chest pain radiating to the left arm, "
        "diaphoresis, and shortness of breath."
    ),
    verbose=True,
)
```

The end-to-end demo cell at the bottom is gated behind `ENABLE_DEMO` (off by default); set it to `True` to run one full case (~150 s, real API tokens).

> **Note:** the notebook currently uses absolute paths near the top (`DATA_DIR`, etc.). Adjust them to your environment before running.

## Data

All evaluation and retrieval data derive from **MIMIC-IV-Note** and **MIMIC-IV-ED**, which are credentialed-access resources on PhysioNet. **No patient data is included in this repository** — the PhysioNet Data Use Agreement prohibits redistribution. To reproduce the evaluation, obtain credentialed access to [MIMIC-IV-Note](https://physionet.org/content/mimic-iv-note/2.2/) and [MIMIC-IV-ED](https://physionet.org/content/mimic-iv-ed/2.2/) and point the notebook's data paths at your local copies.

## Citation

If you use TriAgent or this code in your research, please cite:

```bibtex
@Article{ai7060230,
AUTHOR = {Ibrahim, Ahmed and AlSanousi, Ali and Serag, Ahmed},
TITLE = {TriAgent: An Adaptive Multi-Agent Architecture for Crisis Clinical Decision Support Under Incomplete Information},
JOURNAL = {AI},
VOLUME = {7},
YEAR = {2026},
NUMBER = {6},
ARTICLE-NUMBER = {230},
URL = {https://www.mdpi.com/2673-2688/7/6/230},
ISSN = {2673-2688}
```

## Acknowledgements

Built with [LangGraph](https://github.com/langchain-ai/langgraph) and [LangChain](https://github.com/langchain-ai/langchain). Evaluation data from the [MIMIC-IV](https://physionet.org/content/mimiciv/) family (MIMIC-IV-Note, MIMIC-IV-ED) on PhysioNet. Drug-safety signals from NLM **RxNav** and **OpenFDA**.

## License

Released under the [Creative Commons Attribution 4.0 International (CC BY 4.0)](http://creativecommons.org/licenses/by/4.0/) license.
