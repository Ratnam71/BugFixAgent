# BugFixAgent
# 🤖 BugFixAgent — Autonomous Bug Reproduction & Fix Agent

> **Google ADK-Powered Multi-Agent System for Automated Bug Lifecycle Management**

[![Python](https://img.shields.io/badge/Python-3.11+-blue?logo=python)](https://python.org)
[![Google ADK](https://img.shields.io/badge/Google-ADK-orange?logo=google)](https://developers.google.com/adk)
[![Gemini](https://img.shields.io/badge/LLM-Gemini_Pro-purple?logo=google)](https://deepmind.google/technologies/gemini/)
[![Docker](https://img.shields.io/badge/Docker-SDK-2496ED?logo=docker)](https://www.docker.com/)
[![License](https://img.shields.io/badge/License-MIT-green)](LICENSE)
[![Status](https://img.shields.io/badge/Status-POC_In_Progress-yellow)]()


## 🎯 Overview

**BugFixAgent** is an autonomous, multi-agent AI system that handles the entire bug lifecycle — from a Jira ticket to a merged GitHub Pull Request — with **zero manual intervention**.

| Metric | Before | After |
|--------|--------|-------|
| ⏱ Avg. bug resolution time | 4–23 hours | **< 30 minutes** |
| 🔧 Manual env setup | Required | **Zero** |
| 🧪 Test generation | Manual | **Auto-generated** |
| 📝 PR creation | Manual | **Fully automated** |
| 📊 Audit trail | Inconsistent | **Full trace log** |

---

## 🔥 The Problem

Bug fixing is one of the most expensive and repetitive tasks in software development:

- **2–4 hrs** — Environment setup and bug reproduction from vague tickets
- **3–6 hrs** — Root cause analysis through logs, stack traces, and code
- **2–8 hrs** — Implementing the fix, avoiding regressions, writing tests
- **Days** — PR review cycles, rework, and merge in large teams

> 💡 Average time to fix a single bug: **4–23 hours**. BugFixAgent targets **< 30 minutes**.

---

## ⚡ How It Works

```
Jira Ticket
     │
     ▼
┌─────────────┐
│ Jira Parser │  ──→  Extracts structured BugReport (title, stack trace, repo, branch)
└─────────────┘
     │
     ▼
┌─────────────┐
│  Env Setup  │  ──→  Clones repo, detects stack, builds Docker container
└─────────────┘
     │
     ▼
┌─────────────┐
│  Reproducer │  ──→  Generates test case, runs it, confirms failure (retry 3x)
└─────────────┘
     │
     ▼
┌─────────────┐
│ Root Cause  │  ──→  LLM + RAG analysis of logs, call stack, and similar past bugs
└─────────────┘
     │
     ▼
┌─────────────┐
│ Fix & Test  │  ──→  Generates patch, runs regression suite, iterates up to 5x
└─────────────┘
     │
     ▼
┌─────────────┐
│  PR Agent   │  ──→  Creates branch, commits fix, raises PR, notifies team
└─────────────┘
     │
     ▼
GitHub PR + Jira Updated + Slack Notification
```

---

## 🏗 Architecture

```
┌─────────────────────────────────────────────────────────────┐
│              ORCHESTRATOR AGENT (Google ADK)                │
│         Master Controller · State · Retry · Escalation      │
└──────────────────────┬──────────────────────────────────────┘
                       │  ADK Message Bus
    ┌──────────────────┼──────────────────────────┐
    │          │       │        │         │        │
    ▼          ▼       ▼        ▼         ▼        ▼
 Jira      Env     Repro-   Root     Fix &    PR
Parser    Setup    ducer    Cause    Test    Agent
Agent     Agent    Agent    Agent    Agent
    │          │       │        │         │        │
    └──────────┴───────┴────────┴─────────┴────────┘
                       │
         ┌─────────────▼──────────────┐
         │        TOOLS LAYER         │
         │  GitHub · Jira MCP ·       │
         │  Docker · Pytest/JUnit ·   │
         │  Gemini LLM · Shell ·      │
         │  Vector Store (ChromaDB)   │
         └────────────────────────────┘
                       │
    ┌──────────────────▼──────────────────────┐
    │  State Store · Audit Logs · Slack/PD    │
    └─────────────────────────────────────────┘
```

---

## 🤖 Agent Breakdown

### 1. Jira Parser Agent
- Consumes Jira webhook payload (via MCP)
- Extracts: bug title, description, affected module, stack traces, environment
- Maps ticket to GitHub repo and branch
- Creates a structured `BugReport` object for downstream agents

### 2. Env Setup Agent
- Detects language and framework from `Dockerfile`, `package.json`, `pom.xml`, `requirements.txt`, `.csproj`
- Clones the repo and builds an isolated Docker container
- Injects test credentials via HashiCorp Vault
- Validates environment health before proceeding

### 3. Reproducer Agent
- Generates a targeted test case using Gemini from the bug description
- Runs the test inside the Docker container
- Captures failure output, logs, and stack traces
- Retries up to 3x; escalates if unreproducible

### 4. Root Cause Agent
- Uses RAG to retrieve only the relevant code chunks (not the full codebase)
- Feeds stack trace + retrieved context into Gemini for LLM-powered reasoning
- Cross-references with recent git commits touching the affected files
- Queries vector store for similar historical bugs and past fixes
- Outputs: root cause hypothesis + confidence score + affected file list

### 5. Fix & Test Agent
- Generates a targeted code patch using Gemini Code model
- Applies patch and runs the full regression suite
- Runs SonarQube static analysis as a quality gate
- Writes a regression test to prevent re-occurrence
- Iterates up to 5 times; escalates to human via Slack on failure

### 6. PR Agent
- Creates a feature branch: `fix/<ticket-id>-auto-fix`
- Commits with detailed message referencing the ticket
- Raises a GitHub PR with full description, root cause summary, and test results
- Tags reviewers via CODEOWNERS / git blame
- Updates Jira ticket status and sends Slack notification

---

## 🧠 Project Knowledge Layer

A dedicated knowledge layer provides project-scoped context so agents never operate blind.

### Technology Discovery
Automatically detects the project's tech stack by analyzing:
```
Dockerfile · package.json · requirements.txt
pom.xml · .csproj · build.gradle
GitHub Actions · Jenkinsfile · CI/CD pipelines
```
Loads appropriate debugging strategies for: `.NET` · `Java` · `Python` · `Node.js` · `React` · `Angular`

### RAG-Based Knowledge Base
Built from MCP integrations and indexed into a vector store:

| Source | What's Indexed |
|--------|---------------|
| GitHub (MCP) | Source code, PR history, commit messages, git blame |
| Jira (MCP) | Historical tickets, resolution notes, similar past bugs |
| Confluence (MCP) | Architecture docs, ADRs, runbooks, coding standards |
| Repo scan | README, module structure, function/class signatures |

> **Key principle:** During bug analysis, agents do NOT scan the full codebase. They use the Jira context, stack traces, and code retrieval to fetch only the relevant modules and recent changes.

---

## 🛠 Tech Stack

| Layer | Technology |
|-------|-----------|
| **Agent Framework** | Google ADK (Agent Dev Kit) |
| **Language** | Python 3.11+ |
| **LLM** | Gemini Pro + Gemini Code |
| **RAG / Embedding** | LangChain, LlamaIndex, Google Embeddings |
| **Vector Store** | ChromaDB / pgvector / Weaviate |
| **Code Parsing** | tree-sitter, LSP |
| **Issue Management** | Jira via MCP Server |
| **Source Control** | GitHub REST API, PyGithub, GitPython |
| **Containerization** | Docker SDK for Python, docker-compose |
| **Testing** | Pytest (Python), JUnit (Java), Coverage.py |
| **Static Analysis** | SonarQube |
| **Secrets** | HashiCorp Vault |
| **Notifications** | Slack Bot API, PagerDuty |
| **Observability** | Google Cloud Logging, Prometheus, ADK Trace Viewer |
| **Orchestration** | Kubernetes (optional, production scale) |

---

## 🚀 Getting Started

### Prerequisites

- Python 3.11+
- Docker & Docker Compose
- Google Cloud account with Gemini API access
- Jira instance with MCP server configured
- GitHub account + Personal Access Token

### Installation

```bash
# Clone the repository
git clone https://github.com/your-org/bugfix-agent.git
cd bugfix-agent

# Create and activate virtual environment
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Install Google ADK
pip install google-adk google-generativeai
```

### Quick Start

```bash
# Copy environment config
cp .env.example .env

# Edit .env with your API keys and configuration
nano .env

# Initialize the knowledge base for your project
python -m knowledge.indexer --repo-url https://github.com/your-org/your-repo

# Start the agent system
python -m orchestrator.main
```

### Trigger a Bug Fix

```bash
# Via CLI (provide a Jira ticket ID)
python -m cli.trigger --ticket PROJ-1234

# Via Jira Webhook (configure your Jira to POST to this endpoint)
python -m api.webhook_server  # starts on port 8080
```

---

## ⚙️ Configuration

### `.env` file

```env
# Google Gemini
GOOGLE_API_KEY=your_gemini_api_key
GEMINI_MODEL=gemini-pro
GEMINI_CODE_MODEL=gemini-1.5-pro

# Jira MCP
JIRA_MCP_SERVER_URL=https://mcp.atlassian.com/jira
JIRA_SITE_URL=https://your-org.atlassian.net

# GitHub
GITHUB_TOKEN=ghp_your_token_here
GITHUB_ORG=your-org

# Docker
DOCKER_HOST=unix:///var/run/docker.sock

# Vector Store (ChromaDB)
CHROMA_HOST=localhost
CHROMA_PORT=8000

# Secrets (Vault)
VAULT_URL=http://localhost:8200
VAULT_TOKEN=your_vault_token

# Slack
SLACK_BOT_TOKEN=xoxb-your-slack-token
SLACK_ESCALATION_CHANNEL=#bug-agent-alerts

# Agent Config
MAX_FIX_ATTEMPTS=5
MAX_REPRODUCE_ATTEMPTS=3
```

### Project → Repo Mapping

Edit `config/project_mapping.yaml`:

```yaml
projects:
  PROJ:
    repo: your-org/your-repo
    default_branch: main
    language: python
    test_command: pytest
  FRONTEND:
    repo: your-org/frontend-app
    default_branch: develop
    language: javascript
    test_command: npm test
```

---


For questions, demo requests, or architecture discussions — raise an issue or reach out via Slack at `#bugfix-agent`.

> **Status:** POC in progress — Orchestrator and core agent scaffolding underway. Demo targeting Week 8 on 3 real bug scenarios.
