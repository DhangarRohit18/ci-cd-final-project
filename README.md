# 🚀 CI/CD Final Project: Tools and Practices

## Project Overview

This repository contains the completed Final Project for the Coursera course **CI/CD Tools and Practices** (part of the IBM DevOps and Software Engineering Professional Certificate).

It implements an automated Continuous Integration and Continuous Delivery pipeline utilizing:
- **GitHub Actions** for automated linting (`flake8`) and unit testing (`nose`).
- **Tekton (OpenShift Pipelines)** for cluster-native tasks (cleanup, unit tests with nose, image building with `buildah`, and deployment to OpenShift).
- **Red Hat OpenShift** container platform for deployment, persistent storage via PVC, and live application logging.

---

## Deliverables & Tasks Overview

| Task No. | Rubric Item | File / Deliverable | Status |
| :--- | :--- | :--- | :--- |
| **Task 1** | Project Name & Overview | [README.md](https://github.com/PremKLodhia/ci-cd-final-project/blob/main/README.md) | ✅ Passed |
| **Task 2** | GitHub Actions Workflow (flake8 & nose) | [.github/workflows/workflow.yml](https://github.com/PremKLodhia/ci-cd-final-project/blob/main/.github/workflows/workflow.yml) | ✅ Passed |
| **Task 3** | Tekton Tasks (cleanup & nose) | [.tekton/tasks.yml](https://github.com/PremKLodhia/ci-cd-final-project/blob/main/.tekton/tasks.yml) | ✅ Passed |
| **Task 4** | OpenShift PersistentVolumeClaim Details | `oc-pipelines-console-pvc-details.png` | ✅ Passed |
| **Task 5** | GitHub Actions Workflow Terminal Logs | GitHub Actions Workflow Output | ✅ Passed |
| **Task 6** | OpenShift Pipeline Structure Details | `oc-pipelines-oc-final.png` | ✅ Passed |
| **Task 7** | OpenShift Pipeline Success (Green) Details | `oc-pipelines-oc-green.png` | ✅ Passed |
| **Task 8** | OpenShift Live Application Logs Output | Application Service Logs | ✅ Passed |

---

## ⚙️ CI Pipeline Configuration (GitHub Actions)

File: `.github/workflows/workflow.yml`

```yaml
name: CI workflow

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    container: python:3.9-slim

    steps:
      - name: Checkout
        uses: actions/checkout@v3

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt

      - name: Lint with flake8
        run: |
          flake8 service --count --select=E9,F63,F7,F82 --show-source --statistics
          flake8 service --count --max-complexity=10 --max-line-length=127 --statistics

      - name: Run unit tests with nose
        run: nosetests -v --with-spec --spec-color --with-coverage --cover-package=app
```

---

## 🛠️ Tekton Pipeline Tasks

File: `.tekton/tasks.yml`

### 1. Workspace Cleanup Task
```yaml
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: cleanup
spec:
  description: This task will clean up a workspace by deleting all the files.
  workspaces:
    - name: source
  steps:
    - name: remove
      image: alpine:3
      env:
        - name: WORKSPACE_SOURCE_PATH
          value: $(workspaces.source.path)
      workingDir: $(workspaces.source.path)
      securityContext:
        runAsNonRoot: false
        runAsUser: 0
      script: |
        #!/usr/bin/env sh
        set -eu
        echo "Removing all files from ${WORKSPACE_SOURCE_PATH} ..."
        if [ -d "${WORKSPACE_SOURCE_PATH}" ] ; then
          rm -rf "${WORKSPACE_SOURCE_PATH:?}"/*
          rm -rf "${WORKSPACE_SOURCE_PATH}"/.[!.]*
          rm -rf "${WORKSPACE_SOURCE_PATH}"/..?*
        fi
```

### 2. Nose Unit Tests Task
```yaml
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: nose
spec:
  workspaces:
    - name: source
  params:
    - name: args
      description: Arguments to pass to nose
      type: string
      default: "-v"
  steps:
    - name: nosetests
      image: python:3.9-slim
      workingDir: $(workspaces.source.path)
      script: |
        #!/bin/bash
        set -e
        python -m pip install --upgrade pip wheel
        pip install -r requirements.txt
        nosetests $(params.args)
```

---

## 📷 Screenshots

- **PersistentVolumeClaim Details**: `screenshots/oc-pipelines-console-pvc-details.png`
- **OpenShift Pipeline Structure**: `screenshots/oc-pipelines-oc-final.png`
- **OpenShift Pipeline Success**: `screenshots/oc-pipelines-oc-green.png`
- **Application Logs**: `screenshots/oc-pipelines-app-logs.png`

---

## Author
**Prem Lodhia**  
GitHub: [@PremKLodhia](https://github.com/PremKLodhia)
