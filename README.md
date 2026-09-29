# AMD AI Academy Challenge: Mini-Challenges 2 and 3

Solutions to the mini-challenges of the [lablab.ai × AMD AI Academy Challenge](https://lablab.ai/ai-hackathons/amd-lablab-ai-academy-challenge),
in one repository. Each project is self-contained in its own folder, with its own tests, CI and README.

| Mini-challenge | Folder | What it does |
|---|---|---|
| **MC2: OCR** (licence plates and road signs) | [`mc2-ocr/`](mc2-ocr) | `app.py --input-image PATH` reads a PNG, JPEG or TIFF and writes `{"text", "confidence"}`. A vision model reads four pre-processed views; deterministic code applies the transcription rules and votes. |
| **MC3: RAG** (document Q&A) | [`mc3-rag/`](mc3-rag) | `app.py --index` once, then `--query-id ... --query ...` per question. Answers with the value only and cites the exact set of files, refusing when the corpus does not support an answer. |

Both projects have two entry points: the original service (FastAPI, for people and dashboards) and a
separate **contract mode** (`app.py`) that follows the grader's invocation model. Contract mode is a
thin subpackage (`src/tabula_*/contract/`); the design reasons are in each project's `docs/adr/`.

## Layout

```
mc2-ocr/    Dockerfile.submission  app.py  src/tabula_ocr/{,contract/}  tests/  scripts/  docs/adr/
mc3-rag/    Dockerfile.submission  app.py  src/tabula_rag/{,contract/}  tests/  scripts/  docs/adr/
.github/workflows/   one CI workflow per project (lint, strict types, tests, self-check, Dockerfile lint)
```

## Build a submission image

The grader requires the mandated ROCm base image, at most 60 GiB uncompressed, and the container must use
1-48 GiB of VRAM. Build on a Linux machine with Docker and an AMD GPU, from inside the project folder:

```bash
cd mc2-ocr    # or mc3-rag
docker build -f Dockerfile.submission -t <registry>/<name>:<tag> .
bash scripts/check_submission.sh <registry>/<name>:<tag> <images_dir | corpus_dir>
```

Do not squash the image (it would break the base-image layer check). Do not commit registry references or
credentials to this repository.

## What is and is not verified

Both contract paths are covered by unit tests, and each project's `scripts/contract_selfcheck.py` replays the
grader's process model (real subprocesses, real HTTP) against a **scripted** model, scoring full marks in CI.
That proves parsing, image and file handling, retrieval, verification, timing and the output-file contract.
It does **not** measure how well a real vision-language model reads characters or follows the citation rule,
and the submission images have not yet been run on real ROCm hardware. Run
`scripts/contract_selfcheck.py --live` on the target machine with the organisers' sample data before relying
on any score.

## License

MIT (see each project's `LICENSE`).
