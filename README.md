# MentionDock

Find Instagram and TikTok creators by handle, hashtag, or topic, then let your AI compare and analyze the returned profiles and recent stored posts.

[Website](https://mentiondock.com) · [CLI on npm](https://www.npmjs.com/package/mentiondock) · [On skills.sh](https://skills.sh/pietrastudio/mentiondock-skill/mentiondock) · [Security and permissions](SECURITY.md)

This repository distributes the MentionDock Skill and agent integration metadata, with an explicit boundary for handling untrusted creator content. It contains no executable scripts, hooks, or MCP servers; the separately installed CLI performs login, data queries, and billing actions.

This branch prepares API-key support for an upcoming MentionDock CLI release. Its Skill requires that matching release. Installation instructions and version metadata remain pinned to published `mentiondock@0.1.2` until that release is available; this candidate is not ready for public installation.

## Requirements

- Node.js **22.20.0 or newer** and npm.
- An agent that supports Skills and can execute local CLI commands.
- Network access to MentionDock, plus a browser for Google sign-in and Stripe checkout.
- A MentionDock account and credits for data queries. Installing the Skill does not purchase credits or start a subscription.

## Install the CLI

Install the matching CLI once, before adding the Skill:

```sh
npm i -g mentiondock@0.1.2 --ignore-scripts
mentiondock --version
```

The version command should print `0.1.2`. Ensure npm's global command directory is on your `PATH`.

`--ignore-scripts` skips the CLI package's automatic Skill installation. Choose one of the following installation methods to let your agent manage its own copy of the Skill.

## Add the Skill

### Skills CLI

For Codex and other agents supported by the [Skills CLI](https://github.com/vercel-labs/skills):

```sh
npx skills add pietrastudio/mentiondock-skill --skill mentiondock
```

Choose the agent and installation scope in the installer. Review the [Skill instructions](skills/mentiondock/SKILL.md) before installation.

### Claude Code

Add MentionDock's publisher-maintained marketplace, then install the plugin inside Claude Code:

```text
/plugin marketplace add pietrastudio/mentiondock-skill
/plugin install mentiondock@mentiondock
```

This is MentionDock's own marketplace. Availability through this installation method does not mean the plugin has been listed in Anthropic's official directory or certified by Anthropic.

### Gemini CLI

Run this command in your terminal:

```sh
gemini extensions install https://github.com/pietrastudio/mentiondock-skill
```

Restart Gemini CLI after installation. The extension exposes the same `skills/mentiondock/SKILL.md` through Gemini CLI's [extension format](https://geminicli.com/docs/extensions/reference/#agent-skills). It installs no MCP server or hooks.

## Use MentionDock

Start a new agent session after installing, then ask:

> Use MentionDock to find 10 skincare creators on Instagram with 10,000 to 100,000 followers, and compare their recent stored posts.

The agent checks your saved connection. With the candidate CLI, an available API key can connect directly with `mentiondock login --api-key "<key>"`; omitting the key opens a hidden prompt. Google sign-in remains available in your browser:

```sh
mentiondock login
mentiondock account
mentiondock help
```

Data queries use existing credits. Credit purchases and subscriptions require your authorization and confirmation on Stripe's checkout page. A subscription continues until renewal is canceled; removing the Skill or CLI does not cancel it.

## What you can inspect

- [The Skill](skills/mentiondock/SKILL.md): the complete instructions your agent receives.
- [The published CLI source](https://www.npmjs.com/package/mentiondock/v/0.1.2?activeTab=code): the code that runs locally.
- [Security and permissions](SECURITY.md): local files, network destinations, login, payment, and the scope of available checks.

All integrations load the same Skill. A supported installation format, a directory listing, and a security assessment are different things; this repository makes no claim of platform security certification.

## License

Proprietary. The public repository makes the integration available for inspection and installation with MentionDock. No open-source license is granted for the Skill, CLI, service, or brand assets.
