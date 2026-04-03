---
layout: post
title: "Day 2: Root Syncs, Proxies, and Opening the UI"
subtitle: "Wiring up execution approvals, Git backups, and a Load Balanced dashboard"
---

If Day 1 was about getting the agent breathing, Day 2 was about giving it hands and giving me a dashboard.

The first bottleneck was execution approvals. OpenClaw’s security model blocks destructive commands by default. To unblock the agent, I bound the `execApprovals` client directly to Telegram. Now, when the agent needs to touch the OS, it pushes an interactive approval card directly to my phone.

With permissions sorted, I ran OpenClaw’s native `healthcheck` skill. We audited the GCP host, verified the `ufw` state (inactive, relying instead on the GCP VPC firewall), and triggered `security audit --fix` to lock down local chat histories (`chmod 700`) and strip unrecognized commands from the deny-list.

Next came disaster recovery. I initialized a Git repository at the `~/.openclaw` root. We hit a momentary snag with embedded `.git` modules in the workspace, but wiped it and rebuilt it with a draconian `.gitignore` to strip out `.env`, `credentials/`, and SQL logs. I scheduled a nightly OpenClaw cron job for 2:00 AM that runs a config dump, commits the diff, and pushes the backup to a private repo via SSH.

The crowning achievement of the day was the Control UI. OpenClaw’s dashboard defaults to `localhost`. Rather than messing with SSH port forwarding, I architected a proper entry point via a custom subdomain.
The routing is handled via a GCP HTTPS Load Balancer. The security layer is a GCP Cloud Armor policy restricting access exclusively to a strict whitelist of known, static IP locations. On the VM side, I updated OpenClaw’s `gateway.bind` to `lan`, injected the GCP Health Check IP ranges into `gateway.trustedProxies`, and patched the CORS policy.

The load balancer flipped green. The UI loaded. I authenticated via CLI token generation, paired my browser as an operator, and gained full visual control of the system.

**Daily Telemetry**
*   **Agents Active:** 1 (Winston)
*   **Cronjobs:** 2 (GovCon Daily News, Nightly Git Sync)
*   **Primary Model:** Gemini 3.1 Pro Preview
*   **Tools Exercised:** fs.edit, sys.exec, web.search, cron.add, process.poll
*   **Token Usage:** 4.3M (1.1M Input / 41.9K Output / 3.2M Cached)
*   **Compute Cost:** $3.31
