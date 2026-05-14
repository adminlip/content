---
title: 'Run Omni and Claude Engineers in Daytona'
description: 'Learn how to start Omni Engineer and Claude Engineer in reproducible Daytona workspaces with
devcontainers.'
date: 2026-05-14
author: 'adminlip'
tags: ['daytona', 'devcontainer', 'ai', 'python']
---

# Run Omni and Claude Engineers in Daytona

AI coding agents are most useful when every developer can start them from the same clean environment. Local
setup can still get in the way: one laptop may have a different Python version, another may miss system
packages, and API keys can end up scattered across shell profiles. A Daytona workspace removes that friction
by giving the project a reproducible cloud development environment that is defined in the repository.

This article shows how to run [Omni Engineer](https://github.com/Doriandarko/omni-engineer) and [Claude
Engineer](https://github.com/Doriandarko/claude-engineer) inside Daytona. Both projects now have proposed
`devcontainer.json` files so the workspace can install dependencies, prepare a virtual environment, and copy
the sample environment file during creation. The related devcontainer contributions are open in [Omni Engineer
PR #31](https://github.com/Doriandarko/omni-engineer/pull/31) and [Claude Engineer PR 255](https://github.com/Doriandarko/claude-engineer/pull/255).

## Why Use Daytona for AI Engineering Agents

Omni Engineer and Claude Engineer are Python projects that connect to external model APIs. That makes them
good candidates for Daytona because the important setup steps can be captured once and repeated for every
workspace. Instead of asking contributors to manually install packages and remember which interpreter to use,
the devcontainer prepares a Python 3.12 environment and installs the project dependencies automatically.

Daytona also helps teams separate experiments from their local machines. You can open a workspace, add
temporary API keys to a local `.env` file, test prompts or tool calls, and delete the workspace when the
evaluation is finished. The repository remains the source of truth, while the workspace provides an isolated
place to run the agent.

The workflow in this article follows the same pattern for both repositories:

- Open the repository in Daytona.
- Let the devcontainer create the Python environment.
- Add the required API keys to `.env`.
- Start the agent from the workspace terminal.
- Verify the agent can answer a simple development prompt.

## Prerequisites

Before starting, make sure you have access to Daytona and can create a workspace from a GitHub repository. You
also need API keys for the agent you want to run.

For Omni Engineer, prepare an OpenRouter API key because the repository includes `OPENROUTER_API_KEY` in
`.env.example`.

For Claude Engineer, prepare an Anthropic API key. The repository also lists `E2B_API_KEY` as optional in
`.env.example`, so you only need it if you plan to use E2B-powered code execution features.

You do not need to install Python or project dependencies on your laptop. Daytona builds the workspace from
the devcontainer definition and runs the install commands inside the workspace.

## Start Omni Engineer in a Daytona Workspace

Open Daytona and create a new workspace from the Omni Engineer repository:

```bash
https://github.com/Doriandarko/omni-engineer
```

When Daytona detects the devcontainer configuration from the related contribution, it uses the Python 3.12
devcontainer image. The `postCreateCommand` creates a `.venv` virtual environment, upgrades `pip`, installs
`requirements.txt`, and copies `.env.example` to `.env` if the file does not already exist.

After the workspace finishes building, open the terminal and confirm the virtual environment exists:

```bash
ls -la .venv
```

Next, edit `.env` and add your OpenRouter key:

```bash
OPENROUTER_API_KEY="your_openrouter_api_key"
```

Then activate the virtual environment and run the project:

```bash
source .venv/bin/activate
python main.py
```

Use a small first prompt to verify the agent can read the workspace and respond. For example, ask it to
summarize the repository structure or explain where dependencies are defined. A simple verification prompt
avoids changing files before you know the environment is working.

## Start Claude Engineer in a Daytona Workspace

Create another Daytona workspace from the Claude Engineer repository:

```bash
https://github.com/Doriandarko/claude-engineer
```

The proposed Claude Engineer devcontainer also uses Python 3.12. It installs a few system packages used by
desktop or screenshot-related Python dependencies, creates a `.venv` environment, installs `uv`, installs the
Python requirements, and copies `.env.example` to `.env` when needed.

After the workspace is ready, open `.env` and add your keys:

```bash
ANTHROPIC_API_KEY=your_anthropic_api_key
E2B_API_KEY=your_e2b_api_key
```

If you are not using E2B features, leave `E2B_API_KEY` empty or remove that line. Then activate the
environment and start the application:

```bash
source .venv/bin/activate
python app.py
```

Claude Engineer also includes command-line entry points and alternative scripts in the repository. If you want
the CLI experience instead of the web interface, inspect the repository README in your Daytona workspace and
run the script that matches the mode you want to test.

## Keep Secrets and Workspace State Clean

Do not commit `.env` after adding API keys. Both repositories already use sample environment files to show the
required variables, while the real values should stay in the workspace. If you need to share a reproducible
configuration with teammates, update `.env.example` or documentation instead of committing secrets.

It is also useful to keep AI-generated changes on a separate Git branch. From the workspace terminal, create a
branch before asking an agent to edit code:

```bash
git checkout -b experiment/agent-test
```

This keeps the default branch clean and makes it easier to review the exact diff produced during the agent
session.

## Troubleshooting

If the workspace starts but the agent cannot import a package, rebuild the workspace so the devcontainer
install command runs again. You can also run the install command manually inside the activated environment:

```bash
pip install -r requirements.txt
```

For Claude Engineer, use `uv` if you want to match the devcontainer setup:

```bash
uv pip install -r requirements.txt
```

If the agent reports an authentication error, recheck the variable names in `.env`. Omni Engineer expects
`OPENROUTER_API_KEY`, while Claude Engineer expects `ANTHROPIC_API_KEY` and optionally `E2B_API_KEY`.

If browser or screenshot-related tools fail in Claude Engineer, confirm the workspace was built from the
latest devcontainer contribution. The proposed configuration installs supporting Linux packages such as
`python3-tk`, `scrot`, and `xclip`.

## Conclusion

Running AI engineering agents inside Daytona turns setup into a repeatable repository feature. The
devcontainer defines the Python version, dependency installation, editor extensions, and environment template
copy step. Developers can then focus on testing Omni Engineer or Claude Engineer instead of debugging local
setup.

Use the Omni Engineer workflow when you want to experiment with OpenRouter-backed coding assistance. Use the
Claude Engineer workflow when you want to test Anthropic-backed agent capabilities, including its web
interface and optional E2B integration. In both cases, Daytona gives you a clean workspace that can be
recreated, shared, and removed when the experiment is complete.

## References

- [Omni Engineer repository](https://github.com/Doriandarko/omni-engineer)
- [Claude Engineer repository](https://github.com/Doriandarko/claude-engineer)
- [Omni Engineer devcontainer PR](https://github.com/Doriandarko/omni-engineer/pull/31)
- [Claude Engineer devcontainer PR](https://github.com/Doriandarko/claude-engineer/pull/255)
- [Daytona documentation](https://www.daytona.io/docs/)
