# CI-CD-workflow
# Week 11 – Lecture: A Very Basic CI/CD Pipeline
### Applied AI/ML Course | UET Mardan

---

## Lecture Objective

In the Docker lectures we packaged an app so it runs the same way on every machine.

This lecture answers the next classroom question:

> *"I pushed new code. How do I know the tests still pass — without asking someone to run them by hand?"*

The answer is a **CI pipeline**.

**CI** = Continuous Integration  
**CD** = Continuous Delivery / Deployment

Today we build only the **CI** part: GitHub automatically runs `pytest` every time code is pushed.

By the end of this class you will:

1. Say what CI and CD mean in one sentence each.
2. Run tests on your laptop with `pytest`.
3. Read a GitHub Actions file (`.github/workflows/unittest.yml`).
4. Push to GitHub and watch the pipeline go green or red.

---

## The Problem (Why CI?)

Without CI, the story looks like this:

1. You write `add()` and test it on your laptop. It works.
2. A teammate changes the function and pushes.
3. Nobody runs the tests.
4. The app breaks. You find out later — maybe in class, maybe never.

CI is a **robot classmate** that lives on GitHub.

Every push, it:

1. Takes a clean computer.
2. Installs Python.
3. Installs your packages.
4. Runs `pytest`.
5. Shows a green tick or a red cross.

You do not log into a server. You do not remember the commands. The file in `.github/workflows/` is the recipe.

---

## CI vs CD (keep it this simple)

| Term | Full name | One-sentence meaning | In this lecture |
|------|-----------|----------------------|-----------------|
| **CI** | Continuous Integration | Every push, automatically **test** the code | Yes — we do this |
| **CD** | Continuous Delivery / Deployment | After tests pass, automatically **ship** the app | Not today |

```
  YOU                    GITHUB                         LATER (not today)
  write code  →  push  →  CI: run pytest
                              │
                         pass │ fail
                              ▼
                         CD: deploy   ←  we stop before this step
```

**Rule for this class:** CI asks *"is the code OK?"*  CD asks *"shall we put it online?"*

---

## What's in This Project?

```
CI-CD/
│
├── src/
│   ├── __init__.py
│   └── math_operations.py      ← tiny Python functions (add, subtract, …)
├── tests/
│   ├── __init__.py
│   └── test_math_operations.py ← pytest checks those functions
├── requirements.txt            ← pytest
├── .gitignore
├── .github/
│   └── workflows/
│       └── unittest.yml        ← ⭐ the CI pipeline
└── README.md                   ← this lecture
```

No Flask. No Docker. One idea only: **push → tests run on GitHub**.

---

## How the pipeline works

```
  Your laptop                         GitHub
 ┌────────────┐                      ┌─────────────────────────────┐
 │  git push  │ ───────────────────▶ │  1. Checkout the code       │
 └────────────┘                      │  2. Set up Python           │
                                     │  3. pip install -r reqs     │
                                     │  4. pytest                  │
                                     │         │                   │
                                     │    pass ▼ fail              │
                                     │   green   red               │
                                     └─────────────────────────────┘
```

GitHub starts a **runner** (a temporary Linux computer in the cloud).  
As soon as the `pytest` command runs in that machine, it looks for tests — the same way it does on your laptop.

---

## How to Run This Project

### Prerequisites

- Python 3.10+ on your laptop (3.11 is fine)
- A GitHub account
- `git` installed

### 1. Run the tests locally first

Always do this **before** you push. CI should confirm what you already saw.

```bash
pip install -r requirements.txt
pytest
```

**Expected:** 5 tests passed.

### 2. Put the project on GitHub

Create an empty repository on GitHub (no README, no .gitignore). Then:

```bash
git init
git add .
git commit -m "first commit"
git branch -M main
git remote add origin https://github.com/samadriaz9/CI-CD-workflow.git
git push -u origin main
```

If the repo already exists and you only need to push new files:

```bash
git add .
git commit -m "Add CI lecture and pytest workflow"
git push
```

### 3. Watch the pipeline

1. Open the GitHub repo in a browser.
2. Click the **Actions** tab.
3. Click the latest run (**Unit Tests**).
4. Open the **test** job.

You should see the same four steps as in `unittest.yml`:

1. Checkout code  
2. Set up Python  
3. Install dependencies  
4. Run tests (`pytest`)

Green = tests passed. Red = something failed. Click the red step to read the error.

---

## In-class experiments (do these — this is the lecture)

### Experiment A — Tests on your laptop

```bash
pytest
```

**Expected:** 5 passed.

This is what GitHub will run. If it fails here, it will fail there too.

---

### Experiment B — Watch GitHub run the same command

1. Push the project (`git push`).
2. Open **Actions**.
3. Wait about one minute.

**Expected:** a green tick on the commit.

You did not type `pytest` on GitHub. The workflow file did.

---

### Experiment C — Break a test on purpose (see the red X)

1. Open `src/math_operations.py`.
2. Change `add` so it is wrong:

```python
def add(a, b):
    return a + b + 1   # broken on purpose
```

3. Save. Run locally:

```bash
pytest
```

**Expected:** `test_add` fails. The others still pass.

4. Commit and push. Open **Actions**.

**Expected:** the workflow is **red**. Click **Run tests**. GitHub shows the same assertion error you saw locally.

5. Fix `add` (put it back to `return a + b`). Push again.

**Expected:** green tick.

That is the whole point of CI: a broken push is **visible** immediately.

---

## The workflow file (read this slowly)

File: `.github/workflows/unittest.yml`

GitHub only looks inside `.github/workflows/`.  
Any `.yml` file there is a pipeline.

### When does it run?

```yaml
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
```

- Someone **pushes** to `main` → run.  
- Someone opens a **pull request** into `main` → run.

### What machine?

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
```

`test` is the job name.  
`ubuntu-latest` means a fresh Linux computer. It is empty until our steps fill it.

### The four steps

**Step 1 — Checkout code**

```yaml
- name: Checkout code
  uses: actions/checkout@v4
```

`uses:` means "run a ready-made GitHub Action."  
`checkout` copies your repo onto the runner. Without this, there is no code to test.

**Step 2 — Set up Python** (this is a GitHub Action)

```yaml
- name: Set up Python
  uses: actions/setup-python@v5
  with:
    python-version: "3.11"
```

Installs Python 3.11 on the runner.  
(Older videos use `@v2` and Python `3.8`. The idea is the same.)

**Step 3 — Install dependencies**

```yaml
- name: Install dependencies
  run: |
    python -m pip install --upgrade pip
    pip install -r requirements.txt
```

`run:` means "type these shell commands."  
`|` means more than one line.  
This installs `pytest` from `requirements.txt`.

**Step 4 — Run tests**

```yaml
- name: Run tests
  run: pytest
```

As soon as this command runs on the runner, pytest **looks for tests** (`test_*.py` inside `tests/`) and executes them.

If any test fails, the job fails, and GitHub shows a red X on the commit.

---

## Key words (do not skip)

| Word | Meaning |
|------|---------|
| **Workflow** | The whole YAML file. One pipeline. |
| **Job** | A block of work (`test:`). Can have many steps. |
| **Step** | One task: checkout, setup Python, install, pytest. |
| **Action** | A reusable plugin (`uses: actions/setup-python@v5`). |
| **Runner** | The cloud computer that executes the job. |
| **`on:`** | The trigger (push, pull request, …). |
| **`run:`** | A command you would type in a terminal. |
| **`uses:`** | A ready-made Action instead of writing commands yourself. |

---

## Useful commands

| Command | What it does |
|---------|--------------|
| `pytest` | Run all tests on your laptop |
| `pytest -v` | Same, with extra detail |
| `pytest tests/test_math_operations.py` | Run one test file |
| `git status` | See what you changed |
| `git add .` then `git commit` then `git push` | Send code to GitHub (and start CI) |

On GitHub: **Actions** tab → click a run → click **test** → read the logs.

---

## Comparison: Docker lectures vs this lecture

| | Docker (Lectures 01–03) | This lecture (CI) |
|---|-------------------------|-------------------|
| **Question** | How do we run the app the same way everywhere? | How do we test every push automatically? |
| **Tool** | Docker / Compose | GitHub Actions |
| **Recipe file** | `Dockerfile`, `docker-compose.yml` | `.github/workflows/unittest.yml` |
| **Where it runs** | Your laptop (or a server) | GitHub's runner |
| **Success looks like** | App opens in the browser | Green tick on the commit |

They work together later: CI can build a Docker image after tests pass. That is CD. Not today.

---

## Common mistakes

1. **Workflow file in the wrong folder**  
   It must be `.github/workflows/unittest.yml` (note the dot).  
   `github/workflows/` without the dot will never run.

2. **Tests fail locally, then you push anyway**  
   CI will fail too. Fix `pytest` on your laptop first.

3. **Forgot `requirements.txt`**  
   The runner does not have pytest until you install it.

4. **Pushing to a branch that is not in `on:`**  
   This file only listens to `main`. A push to `dev` will not start the workflow.

5. **Thinking CI deploys the app**  
   This pipeline only runs tests. Nothing goes live.

---

## One-sentence summary

**CI is a workflow file that tells GitHub: when code is pushed, start a clean machine, install Python, install packages, and run pytest.**

- Green tick → the code is safe to keep.  
- Red cross → fix it, then push again.

---

*Happy Learning!*
