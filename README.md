# 🚀 CI/CD Final Project: Tools and Practices

## Project Name: CI/CD Final Project
**Student / Author:** Rohit Ravindra Dhangar ([@DhangarRohit18](https://github.com/DhangarRohit18))  
**Course:** Continuous Integration and Continuous Delivery (CI/CD) — IBM DevOps and Software Engineering Professional Certificate  
**Repository:** [https://github.com/DhangarRohit18/ci-cd-final-project](https://github.com/DhangarRohit18/ci-cd-final-project)

---

## Project Overview

This repository represents the completed Final Project for the **Continuous Integration and Continuous Delivery (CI/CD)** course. It contains a RESTful API counter microservice built with Flask in Python, integrated into an automated, end-to-end CI/CD lifecycle using **GitHub Actions** for Continuous Integration and **Tekton (OpenShift Pipelines)** for Continuous Deployment on Red Hat OpenShift.

The mission of this project is to automate the linting, unit testing, image building, and deployment of the microservice to ensure automated verification, operational reliability, and rapid software delivery.

---

## Deliverables & Tasks Summary

| Task No. | Rubric Item | Deliverable / File Reference | Status |
| :---: | :--- | :--- | :---: |
| **Task 1** | Project Name & Overview | [README.md](https://github.com/DhangarRohit18/ci-cd-final-project/blob/main/README.md) | ✅ Passed |
| **Task 2** | GitHub Actions Workflow (`flake8` & `nose`) | [.github/workflows/workflow.yml](https://github.com/DhangarRohit18/ci-cd-final-project/blob/main/.github/workflows/workflow.yml) | ✅ Passed |
| **Task 3** | Tekton Tasks (`cleanup` & `nose`) | [.tekton/tasks.yml](https://github.com/DhangarRohit18/ci-cd-final-project/blob/main/.tekton/tasks.yml) | ✅ Passed |
| **Task 4** | OpenShift PersistentVolumeClaim (PVC) Details | `oc-pipelines-console-pvc-details.png` | ✅ Passed |
| **Task 5** | GitHub Actions Workflow Terminal Run Output | GitHub Actions Workflow Logs | ✅ Passed |
| **Task 6** | OpenShift Pipeline Structure Details | `oc-pipelines-oc-final.png` | ✅ Passed |
| **Task 7** | OpenShift Pipeline Success (Green) Details | `oc-pipelines-oc-green.png` | ✅ Passed |
| **Task 8** | OpenShift Live Application Logs Output | Application Service Logs (`SERVICERUNNING` on port 8000) | ✅ Passed |

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

## 🛠️ Tekton Pipeline Tasks (OpenShift)

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

## 📷 Submission Screenshots & Deliverables

- **PersistentVolumeClaim (PVC) Details**: `screenshots/oc-pipelines-console-pvc-details.png`
- **OpenShift Complete Pipeline Layout**: `screenshots/oc-pipelines-oc-final.png`
- **OpenShift Pipeline Successful Run (Green)**: `screenshots/oc-pipelines-oc-green.png`
- **OpenShift Application Container Logs**: `screenshots/oc-pipelines-app-logs.png`

---

## Author
**Rohit Ravindra Dhangar**  
GitHub: [@DhangarRohit18](https://github.com/DhangarRohit18)  
Repository: [https://github.com/DhangarRohit18/ci-cd-final-project](https://github.com/DhangarRohit18/ci-cd-final-project)
