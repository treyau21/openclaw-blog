---
layout: post
title: "Day 1: Provisioning, Telegram Pipelines, and the Sandbox"
subtitle: "Deploying OpenClaw on GCP and establishing zero-trust comms"
---

The infrastructure build began today. I provisioned a dedicated GCP Compute instance running Ubuntu 25.10 (`trey-openclaw.internal`) to serve as the host and pulled down the OpenClaw binaries.

Instead of dealing with clunky web interfaces for daily interactions, I wanted a direct, mobile-friendly conduit. I used Telegram's `@BotFather` to mint a fresh bot token and wired it into OpenClaw’s configuration. To ensure the endpoint wasn’t sitting wide open on the internet, I locked the gateway's `allowFrom` boundary strictly to my personal Telegram ID. The interface is now zero-trust by default; the bot silently drops payloads from any other user.

With the comms pipe established, the next requirement was persistent agent state. OpenClaw handles continuity through durable memory files in the local workspace. I initially instructed the agent (named Winston) to adopt a "solutions architect" mindset. I quickly realized I didn't need another engineer analyzing my tasks; I needed a highly efficient personal assistant. I forced a prompt rewrite. The agent dynamically edited its own `SOUL.md` and `USER.md` files to strip the architectural jargon and reset its system persona.

The first real test of the local security boundary happened minutes later. To clean up the workspace, the agent attempted an async `exec` call to `rm BOOTSTRAP.md`. The gateway blocked it instantly, throwing an `approval-timeout`. The execution sandbox works out of the box. Because I hadn't yet mapped an interactive approval client to the gateway's authorization routing, the command died in the queue.

We also hit our first tooling friction. I initiated a `web_search` tool call to sweep my digital footprint. It failed out with a missing Gemini API key error. I attempted to inject it via the CLI, but the config schema validation rejected the key path. Instead of fighting the schema, I bypassed it by injecting the key directly into the host's `~/.openclaw/.env` file and bouncing the gateway service.

State is persistent. The comms pipeline is secure. The sandbox is airtight. Tomorrow, we map the execution approvals to Telegram and harden the perimeter.

**Daily Telemetry**
*   **Agents Active:** 1 (Winston)
*   **Cronjobs:** 0
*   **Primary Model:** Gemini 3.1 Pro Preview
*   **Tools Exercised:** fs.edit, fs.write, sys.exec (denied), web.search (failed)
*   **Token Usage:** 851K (247.2K Input / 16.3K Output / 587.5K Cached)
*   **Compute Cost:** $0.81
