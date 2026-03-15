# MLOps Assignment 4 Report: ML Pipeline Fixes

- **GitHub Repository Link:** [Add your link here]
- **Successful Run Screenshot:** [Add your screenshot here]

## Introduction
This report documents the bugs discovered in the initial GitHub Actions `.yml` file provided, the step-by-step resolution, and the successful implementation of the requested workflow adjustments.

## Identified Bugs and Solutions

### 1. YAML Indentation and Formatting Errors
**Bug:** The initial YAML code completely lacked formatting, starting all elements (such as `push:`, `pull_request:`, `jobs:`, `validate-and-test:`, `steps:`, and individual actions) at the root indentation level.
**How it was solved:** YAML syntax relies heavily on indentation layout to distinguish the hierarchy of data. I reapplied the proper nested structure with standard 2-space increments to correct the logic blocks and actions structure.

### 2. Missing Code Checkout Step
**Bug:** The pipeline jumped straight to Python setup without checking out the workspace code. A runner doesn't instantly have access to `requirements.txt` or code files.
**How it was solved:** Created a "Checkout Repository" step using the `actions/checkout@v4` library directly before the "Set up Python" step, allowing standard file access.

### 3. Missing Actual Code Check in Linter Step
**Bug:** The configuration contained `- name: Linter Check` but no concrete `run:` parameters dictating which linter to load or what folder to lint.
**How it was solved:** Completed the step by introducing `run: flake8 .` to actively scan the codebase via flake8 rules. Added `flake8` to the newly written `requirements.txt` to align with the installation step.

### 4. Incorrect Trigger Source
**Bug:** The trigger was configured to activate via `on: push: branches: main`. This didn't meet the requirement: running on every push *except* `main`.
**How it was solved:** Modified the main block to use the `branches-ignore:` mechanism, explicitly placing `- main` under it.

### 5. Missing Final Artifact Upload Step
**Bug:** A final step to upload the project's documentation (`README.md`) was missing as requested.
**How it was solved:** Added the action `actions/upload-artifact@v4` step at the bottom of the job, pointing to the target local path `README.md` and explicitly naming the output as `project-doc`.

## Final `.github/workflows/ml-pipeline.yml`

```yaml
name: ML Model CI

on:
  push:
    branches-ignore:
      - main
  pull_request:

jobs:
  validate-and-test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4
        
      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.10'

      - name: Install Dependencies
        run: pip install -r requirements.txt

      - name: Linter Check
        run: flake8 .

      - name: Model Dry Test
        run: |
          python -c "import torch; print('Model environment ready!')"
          
      - name: Upload README Artifact
        uses: actions/upload-artifact@v4
        with:
          name: project-doc
          path: README.md
```
