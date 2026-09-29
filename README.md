# AAIB Incident Analysis: Evaluation and Reproduction Guide

This repository provides supplementary artifacts for:

**A Human-centered Intelligent Digital Twin System Integrating AI and Knowledge for Industrial Decision Support**

The artifacts support inspection of the recorded experimental outputs, recalculation of evaluation tables, and fresh execution of the extended evaluation pipeline.

The implementation combines vector retrieval, graph relationships between document chunks, and Corporate Logic Knowledge Memory (CLKM) guidance.

## 1. Supplementary packages

| Archive | Contents |
|---|---|
| `AAIB_Evaluation_Data_v1.zip` | AAIB source materials, provenance records, dataset splits, questions, and initial evaluation documentation. |
| `AAIB_Experiment_v3.zip` | Experimental code, configuration, prompts, graph expansion, and CLKM integration. |
| `AAIB_V3_Semantic_Results.zip` | Recorded model outputs, semantic assessments, supporting evidence, and table-reproduction scripts. |

Extract the archives into separate directories. Run commands from the directory containing the specified Python script.

Use the version 3 experiment package for the extended evaluation. Instructions in the original data package describe an earlier workflow and should not replace the version 3 procedure below.

## 2. Evaluation scope

The prepared dataset contains 18 public AAIB reports and 80 questions:

- Development: five reports and 22 questions.
- Reserved test split: 13 reports and 58 questions.

The reported extended evaluation uses the **development split only**. No held-out test results are reported.

Six configurations are compared:

| Configuration | Components |
|---|---|
| LLM only | Generation without retrieved evidence |
| Vector RAG | Top-five vector retrieval |
| Vector RAG + chunk KG | Vector retrieval followed by graph expansion |
| Vector RAG + chunk KG + static CLKM | Hybrid retrieval with initial guidance |
| Vector RAG + chunk KG + updated CLKM | Hybrid retrieval with updated guidance |
| Vector RAG + updated CLKM | Vector retrieval with the same updated guidance |

The run contains 132 responses: 22 questions × six configurations × one repeat.

The primary semantic analysis uses 21 questions per configuration. Question `aaib-001-q03` is excluded consistently because of conflicting information in its source report. Its outputs remain available for inspection.

Three generation failures are retained with zero credit.

## 3. System requirements

### Checking saved results

Only the following are required:

- Python 3.10 or later.
- A web browser for the answer-review page.
- Storage for the extracted results archive.

No GPU, Ollama installation, model download, or additional Python package is required to recalculate the saved evaluation tables.

### Running new experiments

| Component | Requirement or recommendation |
|---|---|
| Operating system | Windows, Linux, or macOS supported by the installed Ollama version; commands below focus on Windows PowerShell |
| Python | Python 3.10 or later |
| Python dependencies | Standard library for the supplied experiment workflow |
| Model runtime | Ollama with its local API running |
| Generation model | `qwen3-4b` |
| Embedding model | `nomic-embed-text` |
| GPU | A supported GPU is recommended for practical execution time |
| GPU memory | An 8 GB GPU was used during development; this does not guarantee that the entire model and 32K context fit in GPU memory |
| System memory | 32 GB RAM recommended as a practical starting point; actual requirements depend on model placement and context allocation |
| Storage | Allow approximately 20 GB free for software, models, extracted artifacts, and outputs; additional runs require more space |
| Internet | Required for initial software/model downloads and opening external AAIB source links |

The RAM and storage recommendations are planning estimates, not experimentally established minimum requirements.

Partial CPU/GPU execution may occur when GPU memory is insufficient. CPU execution can be substantially slower and may exceed the configured request timeout.

## 4. Install Python and Ollama

### Python

Install Python from:

https://www.python.org/downloads/

On Windows, enable the installer option to add Python to PATH.

Verify installation:

```powershell
python --version
```

### Ollama

Install Ollama using the official instructions:

- Download: https://ollama.com/download
- Windows: https://docs.ollama.com/windows
- Linux: https://docs.ollama.com/linux
- macOS: https://docs.ollama.com/macos
- GPU compatibility: https://docs.ollama.com/gpu

For Windows, install the native application and open a new PowerShell window.

Verify installation:

```powershell
ollama --version
```

Download the required models:

```powershell
ollama pull qwen3-4b
ollama pull nomic-embed-text
ollama list
```

Ollama normally runs in the background after installation on Windows. The experiment connects to:

```text
http://localhost:11434
```

If no Ollama server is running, start one in a separate terminal:

```powershell
ollama serve
```

Do not start a second server if the background application already occupies this address.

Check the local API from PowerShell:

```powershell
Invoke-RestMethod http://localhost:11434/api/version
```

## 5. GPU configuration and checks

For NVIDIA hardware, install a compatible driver and confirm that the GPU is detected:

```powershell
nvidia-smi
```

Ollama manages model placement. No GPU index or CUDA setting needs to be added to the experiment configuration for a normal single-GPU installation.

While a model is loaded, inspect its placement:

```powershell
ollama ps
```

The `PROCESSOR` column indicates whether execution uses GPU, CPU, or a mixture. An empty listing can simply mean that no model is currently loaded.

### Optional single-request configuration

To limit parallel request processing, set the following Windows user environment variable:

```powershell
[Environment]::SetEnvironmentVariable("OLLAMA_NUM_PARALLEL", "1", "User")
```

Fully quit Ollama from the system tray and relaunch it after changing the variable.

This is a resource-management recommendation, not a claim about the recorded run's original server environment.

The experiment specifies the context length through the API's `num_ctx` setting. Changing only Ollama's global context setting does not replace the experiment configuration.

Do not silently reduce context length or output allowance to accommodate hardware. If settings must change, use a separate run directory and document the changes.

## 6. Quick check: reproduce the reported tables

Extract `AAIB_V3_Semantic_Results.zip`.

Open a terminal in the directory containing `reproduce.py`, then run:

```powershell
python reproduce.py
```

This regenerates the evaluation summaries from saved responses and explicit assessment records. It makes no model calls.

To preserve the distributed files for comparison, run this command in a working copy of the extracted results directory.

### Main result files

| File | Purpose |
|---|---|
| `RESULTS.md` | Human-readable tables and interpretation |
| `results.json` | Machine-readable aggregate results |
| `semantic_assessments.jsonl` | Individual scores, requirements, and assessment notes |
| `component_diagnostics.jsonl` | Retrieval and runtime diagnostics |
| `METHODOLOGY.md` | Scoring definitions, exclusions, and limitations |
| `ANSWER_REVIEW.html` | Browser-based inspection of answers and evidence |
| `assessments.py` | Explicit saved semantic judgments used by the aggregation script |
| `inputs/manifest.json` | Recorded experimental settings and backend identity |

Expected primary results, rounded to three decimal places:

| Configuration | Correctness | Meaning completeness | Fully correct and complete | Reasoning: 11 questions | Failures |
|---|---:|---:|---:|---:|---:|
| LLM only | 0.071 | 0.048 | 0/21 | 0.045 | 0 |
| Vector RAG | 0.738 | 0.810 | 12/21 | 0.636 | 0 |
| Vector RAG + chunk KG | 0.786 | 0.810 | 12/21 | 0.636 | 0 |
| Vector RAG + chunk KG + static CLKM | 0.762 | 0.786 | 12/21 | 0.545 | 1 |
| Vector RAG + chunk KG + updated CLKM | 0.810 | 0.845 | 15/21 | 0.636 | 1 |
| Vector RAG + updated CLKM | 0.762 | 0.845 | 14/21 | 0.682 | 1 |

These are normalized semantic rubric scores, not precision, recall, or F1.

Recalculating the tables reproduces the aggregation of the saved judgments. It does not independently reproduce or validate the judgment process.

## 7. Inspect generated answers and evidence

Open `ANSWER_REVIEW.html` in a web browser.

On Windows:

```powershell
Start-Process .\ANSWER_REVIEW.html
```

The page exposes:

- Original generated responses.
- Configuration and question identifiers.
- Generation errors.
- Semantic requirements and assigned scores.
- Assessment explanations.
- Reference passages and links to original AAIB reports.
- Cited source chunks.

The `inputs/` directory also contains raw responses, question and memory snapshots, semantic references, and source chunks.

Reviewers can trace aggregate results to individual responses and examine whether the assigned judgments are supported by the source material.

## 8. Configure a fresh experiment

Extract `AAIB_Experiment_v3.zip` into a new directory.

Open `configs/experiment.json`.

**Required adjustment:** the distributed version 3 configuration defaults to `num_ctx = 16384`, but the reported run used **32768**. Set:

```json
"num_ctx": 32768
```

Verify the principal settings against the saved results manifest:

| Setting | Reported value |
|---|---|
| Generation model | `qwen3-4b` |
| Embedding model | `nomic-embed-text` |
| Chunk size | 250 words |
| Chunk overlap | 40 words |
| Initial vector retrieval | Five chunks |
| Graph expansion seeds | First three vector results |
| Expansion depth | One hop |
| Reranking | None |
| Context window | 32768 tokens |
| Maximum generated output | 1536 tokens |
| Evidence-plus-guidance word limit | 6000 |
| Temperature | 0 |
| Seed | 42 |
| Repeats | 1 |

Graph expansion preserves all initial vector results and appends deduplicated linked chunks. CLKM guidance is selected after retrieval and does not alter evidence ranking or membership.

The hybrid configurations can receive more evidence than vector-only retrieval. This comparison is not an equal-token efficiency experiment.

Model tags can change over time. Compare model identities against `AAIB_V3_Semantic_Results/inputs/manifest.json` and record any differences.

## 9. Validate the code and build an index

From the directory containing `experiment.py`, run:

```powershell
python -m unittest discover -s tests -v
```

Build the development index:

```powershell
python experiment.py build --scope dev
```

The builder uses the five development reports and writes the index under `work/index-dev/`.

The three distributed archives do not contain the exact original retrieval index. Rebuilding can produce different graph candidates, so preserve the rebuilt index and its provenance.

The supplied corpus is already included in the experiment package; no manual copying from the data archive is required for this workflow.

## 10. Check retrieval and context before generation

Run:

```powershell
python experiment.py run --split dev --task open --preflight --out work/reviewer-preflight
```

This prepares retrieval payloads and checks their size without generating answers. It still uses Ollama for embeddings.

Inspect:

```text
work/reviewer-preflight/preflight_summary.json
```

Check for blocked payloads before proceeding.

The context screen is conservative and is not an exact model-specific token count. The pipeline preserves selected evidence rather than silently dropping chunks to fit.

## 11. Generate new responses

After checking the preflight, run:

```powershell
python experiment.py run --split dev --task open --out work/reviewer-open-run
```

This runs 22 development questions across six configurations.

Inspect the generated files, including:

```text
work/reviewer-open-run/answers.jsonl
work/reviewer-open-run/manifest.json
```

Use a new output directory when changing models, configuration, prompts, data, or index contents. Resume an interrupted run only with unchanged inputs and settings.

Retain failures and error records. Do not selectively replace unfavorable answers or omit unsuccessful cases from comparisons.

## 12. Assessing newly generated answers

New outputs require a new semantic assessment using the rubric in `METHODOLOGY.md`.

The results package's `reproduce.py` aggregates judgments for the archived responses only. It is not an automatic evaluator for arbitrary new responses.

Do not replace the archived answers with new outputs and reuse their old scores.

The legacy selection-task scorer evaluates a different task and must not be reported as open-answer semantic quality.

For independent validation, assess responses against their source reports and record the assessor, rubric, exclusions, and any adjudication procedure.

## 13. Troubleshooting

| Issue | Suggested check |
|---|---|
| `python` or `ollama` not found | Confirm installation and reopen the terminal |
| Connection refused at port 11434 | Start Ollama and check the local API |
| Address already in use when running `ollama serve` | Check whether the background Ollama application is already running |
| Model not found | Run `ollama list` and download the required model |
| Slow generation | Inspect `ollama ps` for CPU offloading and check available memory |
| Request timeout | Check model loading, memory use, and server logs; document any timeout-setting change |
| Context preflight blocked | Verify `num_ctx = 32768` and inspect payload diagnostics |
| Generation ends at the output limit | Preserve the recorded failure; changing the allowance defines a different run |
| Fresh scores differ from archived scores | Compare model identities, index, prompts, settings, outputs, and assessment decisions |

## 14. Reproducibility boundaries

- This package is a reconstructed research evaluation, not the recovered original industrial implementation.
- The preliminary experiments are not reproduced by this package.
- The reported extended results are development results from five reports.
- Semantic judgments were made using conversational AI with configuration identities visible; they are not independent expert assessments.
- Recalculation reproduces saved-score aggregation, not independent semantic judgment.
- Reference-quote coverage measures availability of selected quoted passages, not exhaustive semantic retrieval recall.
- Static and updated CLKM contain prepared guidance. The experiment does not demonstrate autonomous KG-to-CLKM learning or a complete human feedback cycle.
- Fresh graph extraction and generation can differ across model versions, hardware, and runtime environments, even at temperature zero.
- The reported results do not establish operational maintenance performance or measured benefits to human users.

## 15. Official installation references

- Python: https://www.python.org/downloads/
- Ollama download: https://ollama.com/download
- Ollama Windows installation: https://docs.ollama.com/windows
- Ollama Linux installation: https://docs.ollama.com/linux
- Ollama macOS installation: https://docs.ollama.com/macos
- Ollama GPU support: https://docs.ollama.com/gpu
- Ollama configuration and troubleshooting: https://docs.ollama.com/faq
- AAIB reports: https://www.gov.uk/aaib-reports
