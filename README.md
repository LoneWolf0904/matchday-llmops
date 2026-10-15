# matchday-llmops

A football question-answering assistant (RAG) over the Laws of the Game and Premier League history, built as a vehicle for learning and demonstrating LLMOps: tracing, evaluation, eval-gated CI/CD and AWS deployment.

The app is deliberately simple. The operations around it are the point.

## Status

Work in progress, built in the open. Phase 0 is complete; nothing else is built yet.

| Phase | What | Status |
|---|---|---|
| 0 | Repo, secrets hygiene, secret scanning, Docker basics | Done |
| 1 | RAG app, FastAPI service, Langfuse tracing | Not started |
| 2 | Golden dataset and evaluation suite | Not started |
| 3 | CI/CD with an eval gate that blocks quality regressions | Not started |
| 4 | Deployment on AWS (ECS Fargate, Bedrock, CloudWatch) | Not started |

## What it will do

- Answer questions grounded in two document sets: the IFAB Laws of the Game and Wikipedia's Premier League articles
- Trace every request (tokens, latency, retrieved chunks) with a self-hosted Langfuse
- Score answers against a hand-written golden dataset and block pull requests that make quality worse
- Run as a container on AWS with logs, an alarm and a teardown script

## Planned stack

Python, FastAPI, Chroma, Langfuse, promptfoo and/or Ragas, GitHub Actions, Docker, AWS (ECR, ECS Fargate, Bedrock, CloudWatch). Final choices are confirmed in each phase.

## Security

This is a public repo, so secrets are handled defensively:

- `.env` is gitignored; `.env.example` lists variable names only
- A gitleaks pre-commit hook scans every commit locally
- GitHub secret scanning and push protection are enabled
- A CI secret scan is planned for Phase 3

To enable the local hook after cloning:

```
py -3.12 -m venv .venv
.venv\Scripts\Activate.ps1
pip install pre-commit
pre-commit install
```

## Data and licensing

Source documents are not stored in this repo. A download script will fetch them. Wikipedia text is licensed CC BY-SA and will be attributed. A project licence has not been chosen yet.

## Author

Neel Bathija

