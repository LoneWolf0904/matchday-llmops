# matchday-llmops: Project Scope

A football RAG assistant (Laws of the Game + Premier League history) built as a vehicle for learning and demonstrating LLMOps: tracing, evaluation, eval-gated CI/CD, and AWS deployment.

Owner: Neel Bathija
Repo: `matchday-llmops` (public)
Status: Scoping complete, build not started
Scoped: 14 Oct 2026

---

## 1. Why this project exists

**Primary goal:** a portfolio project that gets me interviews for UK graduate AI engineering, data engineering and software engineering roles.

**Secondary goal:** actually understand LLMOps concepts, not just copy tutorials. I'm starting from "I know nothing" on purpose.

**The pitch (what a recruiter should take away in 30 seconds):**
"Built a RAG application with full LLM observability, an automated evaluation suite that blocks regressions in CI, and containerised deployment to AWS."

**What the app does is deliberately boring.** The app is a vehicle. The ops machinery around it is the product.

---

## 2. How we work (the learning workflow)

Every topic follows the same loop:

1. **Concept**: Claude explains it from scratch, with analogies, visuals, slides or dashboards where useful.
2. **Check**: Claude asks questions or sets a small exercise. I answer in my own words.
3. **Gate**: Claude only moves to the build once I've actually understood. If I haven't, we go round again.
4. **Build**: we implement it together in the repo.
5. **Notes**: a note file for that topic gets written into `/notes` (see section 5).

Rules for the process:

- I assume zero knowledge, even where I've seen something before.
- No fixed timeline. I give it as much time as I can. Priority 1 is still the job search, so this never blocks applications.
- Non-negotiable: at least one full rest day a week.

### End-of-session commit protocol

Whenever I say I'm done for the day, Claude must:

1. Review what changed this session (code, notes, docs, configs).
2. Tell me exactly what needs committing, with suggested commit messages and the exact commands.
3. Check nothing sensitive is in the diff (keys, `.env`, tokens, raw data files).
4. Not let me leave until I've confirmed I've committed and pushed, or told it I'm skipping on purpose.

**Control boundary:** Claude has read-only access to the repo. Claude hands me files and commands. Only I commit and push.

---

## 3. Scope: what the app is

A question-answering assistant over two document sets:

| Source | Type | Why it's here | Licence note |
|---|---|---|---|
| IFAB Laws of the Game (+ VAR protocol docs) | Clean, structured PDFs | Stable, easy to test, teaches chunking on tidy text | Free to read; **check reuse terms before committing the raw PDFs** |
| Wikipedia Premier League articles (seasons, clubs, managers) | Messy, long web articles | Real-world mess, teaches chunking and retrieval properly | CC BY-SA: attribution required |

**Decision:** raw source documents are **not committed** to the repo. A download script fetches them, and the repo documents where they came from. This avoids licence problems and keeps the repo small.

**Out of scope (deliberately):**

- Match reports or news from BBC, Sky, The Athletic etc. (copyrighted)
- Pure stats questions (league tables, xG). That's a SQL or API problem, not RAG.
- A fancy frontend. A simple API plus a minimal UI is enough.
- Fine-tuning models.

---

## 4. Phases

Each phase has: concepts to learn, exercises, what we build, and a deliverable I can point to.

### Phase 0: Setup and foundations

**Concepts:** git workflow basics, virtual environments, `.gitignore`, secrets management (and why public repos make this critical), Docker refresher, project structure.

**Build:**
- Create the public GitHub repo `matchday-llmops`
- `.gitignore`, `.env.example`, README skeleton
- Secret scanning before commit (pre-commit hook with gitleaks or similar)
- Python environment, project layout, "hello world" container that builds and runs
- Copy this scope doc into the repo

**Check:** explain in my own words why `.env` must never be committed and how I'd recover if a key leaked.

**Deliverable:** clean repo, first commits, container runs locally.

### Phase 1: The RAG app + observability

**Concepts (in order):**
1. What RAG is and why it exists (open-book exam analogy)
2. Embeddings: text as meaning-vectors
3. Chunking: size, overlap, structure-aware splitting, and why it's the biggest lever on quality
4. Vector stores and similarity search
5. Prompt construction and grounding ("answer only from the context")
6. What a trace is, and why LLM apps need observability (tokens, latency, cost, retrieved chunks)

**Build:**
- Ingestion pipeline: download, parse, chunk, embed, store
- Retrieval + generation chain
- FastAPI service with a `/ask` endpoint
- Langfuse (self-hosted in Docker) tracing every request
- Dockerise the whole thing, docker-compose for app + Langfuse

**Exercise:** deliberately ask questions the bot gets wrong, then use the traces to diagnose whether retrieval or generation failed.

**Deliverable:** working bot, trace dashboard with real data, architecture diagram v1.

### Phase 2: Evaluation (the heart of the project)

**Concepts:**
1. Why you can't unit test LLM outputs with equality
2. Golden datasets: what makes a good question set
3. Retrieval metrics (did we fetch the right chunk?) vs generation metrics (was the answer right and grounded?)
4. LLM-as-judge: how it works, where it lies, how to sanity-check it
5. Prompt versioning: prompts as files in git, not strings in code
6. Regression testing for prompts and chunking changes

**Build:**
- Golden dataset of 40-50 question/answer pairs, written by me (I know the football), with source references
- Eval harness (promptfoo and/or Ragas, decided when we get there)
- Versioned prompts in `/prompts`
- An eval report I can read at a glance, ideally as a dashboard

**Exercise:** break the prompt on purpose, then show the eval suite catching it. Change chunk size and measure the effect.

**Deliverable:** reproducible eval run, a before/after comparison table, a short written findings section for the README.

### Phase 3: CI/CD

**Concepts:** what CI and CD actually are, GitHub Actions workflow anatomy, secrets in CI, why eval-gated deployment matters, container registries, OIDC (short-lived credentials) vs long-lived keys.

**Build:**
- GitHub Actions: lint + tests on every PR
- Eval suite runs on PRs and **blocks merge if quality drops below a threshold**
- Docker image build in CI
- AWS account setup happens here (see section 6), then push images to ECR

**Exercise:** open a PR with a deliberately bad prompt change and watch the gate reject it. Screenshot it for the README.

**Deliverable:** green pipeline, one documented blocked PR, image in ECR.

### Phase 4: AWS deployment

**Concepts:** IAM basics and least privilege, ECS and Fargate (tasks, services, task definitions), networking basics (VPC, security groups, load balancer), Bedrock for model access, CloudWatch logs and alarms, cost control.

**Build:**
- Run the container on ECS Fargate
- Bedrock for the LLM (and embeddings if it makes sense)
- CloudWatch logs and at least one alarm
- Deploy step wired into the CI/CD pipeline
- Terraform only if time allows (stretch)

**Deliverable:** live endpoint I can demo, architecture diagram v2, a teardown script so it's not burning money when idle.

### Phase 5 (stretch): FPL assistant

**Concepts:** tool calling / function calling, working with live data, data freshness, handling an unofficial API that can break.

**Build:** extend the bot with FPL API tools (prices, injuries, fixtures) plus a weekly refresh job. This is the strongest "ops over time" story, but it's riskier, so it only happens after Phases 0-4 are finished and documented.

---

## 5. Notes system

Notes live in the repo so they're versioned and I can revisit any topic.

```
notes/
  PROGRESS.md              running log: what we did, what's next, open questions
  00-setup.md
  01-rag-fundamentals.md
  02-embeddings-and-chunking.md
  03-observability-and-tracing.md
  04-evaluation.md
  05-cicd.md
  06-aws.md
  glossary.md              every term, one or two lines, in plain English
```

Each topic note contains: the concept in plain words, the analogy used, the exercise and my answer, mistakes I made and what fixed them, and "if I forget this, re-read this part."

---

## 6. AWS plan

- No AWS account yet. Created at the start of Phase 3, not before.
- First actions on the day it's created: enable MFA on the root account, **set a billing alarm**, create a non-root admin user, never create access keys for root.
- GitHub Actions authenticates via OIDC, so no long-lived AWS keys ever sit in the repo or in GitHub secrets.
- Resources are torn down when not demoing. Teardown script is a deliverable, not an afterthought.

---

## 7. Proposed tech stack

Final choices are confirmed in the relevant phase, after the concept lesson, not before.

| Layer | Starting choice | Notes |
|---|---|---|
| Language | Python | Already comfortable |
| API | FastAPI | |
| Vector store | Chroma (local) | Revisit for deployment |
| Embeddings | TBD (Bedrock Titan vs local model) | Decided in Phase 1 |
| LLM | Bedrock (Claude or similar) | Cost vs convenience decided in Phase 1 |
| Tracing | Langfuse (self-hosted) | |
| Eval | promptfoo and/or Ragas | |
| CI/CD | GitHub Actions | |
| Containers | Docker, docker-compose | |
| Cloud | AWS: ECR, ECS Fargate, Bedrock, CloudWatch | |
| IaC | Terraform | Stretch only |

---

## 8. Repo structure (target)

```
matchday-llmops/
  README.md
  SCOPE.md
  .gitignore
  .env.example
  .pre-commit-config.yaml
  docker-compose.yml
  Dockerfile
  pyproject.toml
  src/matchday/        app code (ingest, retrieval, api)
  prompts/             versioned prompt files
  evals/               golden dataset + eval configs
  scripts/             data download, teardown, helpers
  tests/
  notes/               learning notes (section 5)
  docs/                architecture diagrams
  .github/workflows/   CI/CD pipelines
```

---

## 9. Security rules (public repo)

- Never commit: `.env`, API keys, AWS credentials, raw datasets, Langfuse secrets.
- `.env.example` holds variable names only, never values.
- Secret scanner runs as a pre-commit hook and in CI.
- If a secret is ever committed: **rotate it immediately**, then clean history. Deleting the file is not enough, because git remembers.

---

## 10. Definition of done

The project is "done" (and gets added to the CV) when:

- [ ] Public repo with a clear README, architecture diagram and setup instructions
- [ ] RAG app answers questions over both document sets
- [ ] Every request is traced in Langfuse
- [ ] Golden dataset of 40+ questions with a reproducible eval run
- [ ] CI blocks a PR that degrades eval scores (with a documented example)
- [ ] Deployed on AWS with logs and an alarm, plus a teardown script
- [ ] README has a findings section: what I changed, what the evals showed
- [ ] Notes folder complete enough that I could explain every concept in an interview

Phase 5 (FPL) is a bonus. Done does not depend on it.

---

## 11. CV bullet drafts (to finalise at the end, with real numbers)

Format: Problem / Solution / Result.

- Built an end-to-end LLMOps pipeline for a RAG application: Langfuse tracing, an automated evaluation suite over a [N]-question golden dataset, and eval-gated CI/CD that blocks quality regressions before merge.
- Deployed the containerised service to AWS (ECS Fargate, ECR, Bedrock, CloudWatch) with automated deployments via GitHub Actions and OIDC authentication.
- Improved answer accuracy from [X]% to [Y]% by evaluating chunking strategies and prompt versions against a measured baseline.

---

## 12. Risks and honest caveats

- **Scope creep** is the main enemy. A finished Phases 0-4 beats an unfinished Phase 5.
- **AWS cost** is real if things are left running. Billing alarm first, teardown after every demo.
- **Time:** no deadline, but the job search comes first. If applications need attention, this pauses.
- **Eval quality:** a weak golden dataset makes the whole eval story hollow. I write the questions myself, because that's where my football knowledge pays off.
- **LLM-as-judge** can be biased or wrong. We validate it against my own judgement on a sample.

---

## 13. Open questions (to resolve during the build)

1. Embedding model: Bedrock Titan vs local sentence-transformers (Phase 1)
2. Which LLM on Bedrock, and the cost per eval run (Phase 1-2)
3. promptfoo vs Ragas, or both (Phase 2)
4. How the vector index ships to AWS: baked into the image, or loaded from S3 (Phase 4)
5. Whether Terraform is worth the time (Phase 4)

---

## 14. First session checklist (start of the new project)

1. Create the new Claude project and upload this file.
2. Start Phase 0: create the repo, `.gitignore` and secrets setup first.
3. Commit this `SCOPE.md` as the repo's first or second commit.
4. Begin the first concept lesson (RAG fundamentals) with a check question before any code.
