# Trinity: The Sovereign AI Agents Platform
**A Comprehensive Step-by-Step Tutorial & Guide**

Welcome to Trinity! This standalone, comprehensive guide is designed for developers, operators, and end-users who are completely new to Trinity and AI agents. It will walk you through everything from understanding what Trinity is, to installing it from scratch, deploying your first agent, and building a multi-agent system.

---

## Table of Contents

1. [Introduction: What is Trinity?](#1-introduction-what-is-trinity)
2. [Step 1: Installing Trinity from Scratch](#2-step-1-installing-trinity-from-scratch)
3. [Step 2: Building and Deploying Your First Agent](#3-step-2-building-and-deploying-your-first-agent)
4. [Step 3: Setting Up a Multi-Agent System](#4-step-3-setting-up-a-multi-agent-system)
5. [Exploring Other Platform Features](#5-exploring-other-platform-features)
6. [Conclusion & Next Steps](#6-conclusion--next-steps)

---

## 1. Introduction: What is Trinity?

**Trinity** is a sovereign AI agents platform. It is infrastructure designed to orchestrate and run autonomous AI agents in a production environment.

### Why Trinity?
While you can build AI agents on your laptop using tools like Claude Code or Gemini CLI, running them in production requires robust infrastructure. Trinity provides:
- **Isolation:** Each agent runs in its own secure Docker container.
- **Sovereignty:** You run Trinity on your own infrastructure (self-hosted or private cloud). Your data never leaves your perimeter, avoiding SaaS vendor lock-in.
- **Observability:** Real-time monitoring, live dashboards, cost tracking, and audit trails.
- **Orchestration:** Built-in scheduling, agent-to-agent delegation, and human-in-the-loop approvals.
- **Persistent Memory:** Agents remember context across sessions.

### Key Concepts
- **Agents:** Individual AI entities configured via a template (`template.yaml` and `CLAUDE.md`) to perform specific tasks.
- **Control Plane (Web UI):** The central dashboard where operators can view, monitor, and manage the entire fleet of agents.
- **Multi-Agent Systems:** Groups of agents that collaborate via shared folders or direct messaging (via the Model Context Protocol, or MCP) to achieve complex goals.

---

## 2. Step 1: Installing Trinity from Scratch

Installing Trinity is designed to be quick and easy using Docker.

### Prerequisites
Before you start, ensure you have:
1. **Docker and Docker Compose v2** installed and running on your machine (e.g., Docker Desktop for Mac/Windows, or Docker Engine for Linux).
2. At least **5 GB of free disk space**.
3. **API Keys:** An Anthropic API key (for Claude) or a Google API key (for Gemini).

### Installation via Quickstart Script
The easiest way to get started is to use the interactive quickstart script.

1. Open your terminal.
2. Clone the repository and navigate into it:
   ```bash
   git clone https://github.com/abilityai/trinity.git
   cd trinity
   ```
3. Run the quickstart script:
   ```bash
   ./quickstart.sh
   ```
   *Note: For a fully non-interactive setup with auto-generated secrets, you can run `./quickstart.sh --defaults`.*

### Manual Installation (Alternative)
If you prefer to see exactly what is happening, you can install manually:

1. Clone the repository:
   ```bash
   git clone https://github.com/abilityai/trinity.git
   cd trinity
   ```
2. Set up the environment variables:
   ```bash
   cp .env.example .env
   ```
   Open `.env` in a text editor and set at minimum:
   - `SECRET_KEY` (Generate one using: `openssl rand -hex 32`)
   - `ADMIN_PASSWORD` (Set a strong password, at least 8 characters)
   - `ANTHROPIC_API_KEY` (Your API key)
3. Build the base images and start the services:
   ```bash
   ./scripts/deploy/start.sh
   ```

### First-Time Setup in the UI
1. Once the containers are running, open your web browser and go to `http://localhost`.
2. You will be greeted by the setup wizard.
3. Log in with the username `admin` and the `ADMIN_PASSWORD` you configured.
4. Go to **Settings → API Keys** in the UI to confirm or configure your Anthropic API key.

Congratulations! Your Trinity platform is now up and running.

---

## 3. Step 2: Building and Deploying Your First Agent

Now that Trinity is running, let's deploy your first agent. The recommended workflow uses **Claude Code** and the **abilities plugin marketplace**.

### Setup the Toolkit

1. Open your terminal (ensure you have Claude Code installed).
2. Add the marketplace and install the necessary plugins:
   ```bash
   /plugin marketplace add abilityai/abilities
   /plugin install trinity@abilityai
   /plugin install create-agent@abilityai
   ```

### Scaffold Your Agent

Let's create a simple agent from a blank canvas.
1. In your terminal, run the scaffold wizard:
   ```bash
   /create-agent:custom
   ```
   *Follow the prompts to define your agent's name, persona, and initial instructions.*
   This process creates a directory containing the Trinity-compatible files:
   - `template.yaml`: Metadata and configuration.
   - `CLAUDE.md`: The agent's core instructions.
   - `.mcp.json.template`: Tool configuration.

### Deploy to Trinity

1. Connect your local environment to your Trinity instance (you only need to do this once):
   ```bash
   /trinity:connect
   ```
   *Enter your instance URL (`http://localhost`) and follow the email verification.*
2. Deploy the agent you just scaffolded:
   ```bash
   /trinity:onboard
   ```
   This command packages your agent, uploads it to Trinity, and starts the container.

### Interact with Your Agent
1. Open the Trinity Web UI (`http://localhost`).
2. Go to the **Agents** tab. You should see your new agent with a "Running" status.
3. Click on the agent to open the **Chat** interface and say hello!

---

## 4. Step 3: Setting Up a Multi-Agent System

For complex workflows, a single agent isn't enough. A **multi-agent system** coordinates multiple specialized agents.

### Why Multi-Agent?
- **Specialization:** Separate roles (e.g., a researcher agent and a writer agent).
- **Parallelism:** Multiple agents working at the same time.
- **Clear Boundaries:** Avoiding one agent getting confused by having too many instructions.

### The System Manifest Approach
The best way to deploy a multi-agent system is using a single declarative YAML file called a **System Manifest**.

#### Example: Content Production System
Let's define a system with two agents: an *orchestrator* that plans work, and a *worker* that executes it.

1. Create a file named `my-system.yaml` with the following content:

```yaml
name: content-production
description: Autonomous content pipeline

agents:
  orchestrator:
    template: github:abilityai/agent-corbin
    resources: {cpu: "2", memory: "4g"}
    folders: {expose: true, consume: true}
    schedules:
      - name: daily-planning
        cron: "0 9 * * *"
        message: "Plan today's content tasks"

  writer:
    template: github:abilityai/agent-ruby
    folders: {expose: true, consume: true}

permissions:
  preset: orchestrator-workers
```

**What this manifest does:**
- Creates two agents: `content-production-orchestrator` and `content-production-writer`.
- Sets up **Shared Folders** so they can pass files back and forth (e.g., the orchestrator writes a plan to a file, the writer reads it).
- Grants **Permissions** so the orchestrator can securely communicate with the writer via direct messaging.
- Sets a **Schedule** for the orchestrator to run autonomously every day at 9 AM.

#### Deploying the System
You can deploy this manifest via the Trinity API or CLI.

Using the API (from your terminal):
```bash
curl -X POST http://localhost:8000/api/systems/deploy \
  -H "Content-Type: application/json" \
  -d "{\"manifest\": \"$(cat my-system.yaml | sed 's/"/\\"/g' | awk '{printf "%s\\n", $0}')\"}"
```
*(Note: Be sure to set up your authentication headers if your API requires a token).*

Once deployed, go to your Trinity Web UI. You will see both agents running. You can view their interactions in the **Collaboration Dashboard** on the home page!

---

## 5. Exploring Other Platform Features

Trinity is packed with powerful features to manage your autonomous fleet:

### Observability & The Grid
- **Grid View:** See your entire fleet as tiles showing live status, sparklines for activity, and health metrics.
- **Operations Page:** A unified view showing all executions across the fleet, success rates, and an operator queue for tasks requiring human approval.

### Secure Credentials (Hot-Reloading)
Trinity handles secrets securely. Instead of hardcoding API keys, agents use templates (`.mcp.json.template`). Operators store the actual values encrypted in Trinity.
- You can update an agent's credentials on the fly via the API without needing to restart the agent!

### Human-in-the-Loop Approvals
You can configure agents to pause and ask for permission before taking critical actions (like posting to a live social media account).
- The agent pauses execution and puts a request in the **Operator Queue** on the Web UI.
- A human reviews the request and clicks "Approve" or "Deny".

### Integrations & Channels
Trinity agents don't just live in the web UI. You can connect them to:
- **Slack:** Mention your agent in a channel to trigger a workflow.
- **Telegram & WhatsApp:** Chat with agents on mobile.
- **Webhooks:** Trigger an agent's schedule programmatically from external services.

---

## 6. Conclusion & Next Steps

You have now successfully:
1. Learned what Trinity is and why it's powerful.
2. Installed the platform locally.
3. Scaffolded and deployed your first autonomous agent.
4. Designed and deployed a coordinated multi-agent system.

**Where to go next?**
- **Experiment with Skills:** Add custom Python or Bash scripts to your agent's `.claude/skills/` directory.
- **Read the Public Agent Templates:** Explore templates like [Cornelius (Knowledge Base)](https://github.com/Abilityai/cornelius) or [Ruby (Content Creator)](https://github.com/abilityai/agent-ruby) on GitHub to see production-ready agents.
- **Join the Community:** Visit [docs.ability.ai](https://docs.ability.ai) to chat with the built-in documentation agent, or attend the free live workshops hosted by Ability.ai.

Happy building with Trinity!
