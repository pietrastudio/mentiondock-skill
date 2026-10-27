# Security and permissions

This document describes the MentionDock Skill and the matching `mentiondock@0.1.5` CLI. The dated checks below retain their original version scope. This is a permission and behavior disclosure, not a security certification.

## Repository and execution boundary

This repository contains readable Skill instructions and integration metadata. It has no executable scripts, installation hooks, background services, or MCP servers.

The Skill tells an agent how to use the separately installed MentionDock CLI. That CLI runs with the local permissions you grant your agent and terminal. The Skill is not a sandbox, and an agent's confirmation behavior also depends on the agent and its settings.

The [CLI package](https://www.npmjs.com/package/mentiondock/v/0.1.5?activeTab=code) contains JavaScript source that can be inspected before installation.

## Installation and updates

The recommended installation command in this repository uses `npm i -g mentiondock@0.1.5 --ignore-scripts`. It installs the CLI in npm's global directory and skips package lifecycle scripts. Your selected agent or Skill installer then manages the Skill files.

When installed globally **without** `--ignore-scripts`, the CLI's `postinstall` installs default user-level Codex and Claude Code Skills, or refreshes previously registered Skill locations. The interactive `npx mentiondock@latest` setup instead displays the installation plan for confirmation.

The CLI can check npm for a newer version, at most once per day during successful business commands. This check does not install an update. Running `mentiondock update` explicitly installs the release and refreshes Skill locations registered by the MentionDock installer.

CLI-managed Skill installation passes `DISABLE_TELEMETRY=1` and `DO_NOT_TRACK=1` to its bundled Skills installer. Separately invoked tools, including `npx skills add` and agent marketplace clients, follow their own network and telemetry settings.

## Local files

The CLI stores its own state under `~/.mentiondock`:

| Data | Purpose |
| --- | --- |
| `credentials.json` | MentionDock access/refresh tokens for Google sign-in, or an API key, plus account identity, service address, and applicable expiry |
| `login.json` | Pending Google sign-in and a private PKCE verifier |
| `requests/` | Query criteria and request IDs for recovery |
| `purchases/` and `subscriptions/` | Purchase and subscription recovery records |
| `installations.json` and `update-check.json` | CLI-managed Skill locations and version-check cache |

Business state directories are created with Unix mode `0700`, and state files with mode `0600`. Credentials are stored as local JSON, not encrypted in an operating-system keychain. Protect the local user account and avoid sharing these files.

Skill installation also writes the chosen user or project Skill directories and installer lock files. Global CLI installation writes npm's configured global location.

`mentiondock logout` removes the local credentials and pending login. For Google connections it also revokes the current refresh session; existing access tokens retain their expiry. For API-key connections it clears only this computer’s copy, and key revocation remains a separate administrator action. Query and billing recovery records remain; other devices and subscription renewal are unaffected.

## Network and information sent

| Destination | Purpose |
| --- | --- |
| `https://exapi.pietrastudio.com` | MentionDock's default API: authentication, creator queries, account, credits, and billing |
| Your configured npm registry, normally `https://registry.npmjs.org` | Package installation and version checks |
| Google sign-in in your browser | Authorizing your MentionDock account |
| `https://checkout.stripe.com/c/pay/…` in your browser | Confirming a credit-pack purchase or subscription |

Creator queries send their criteria; authenticated requests send the MentionDock access token or API key to the selected service. The CLI does not scan or upload arbitrary project files. Data returned by MentionDock becomes visible to the agent you chose and is governed by that agent's data handling.

An explicit `--service` option or `MENTIONDOCK_URL` environment variable can select another service. The CLI requires an HTTPS origin, allowing HTTP only for local development. Saved credentials and recovery records are bound to their account and service. Authenticated API requests reject redirects.

The installer or your browser may contact additional destinations as part of their own package retrieval, sign-in, or payment flows.

## Sign-in and payment

Google sign-in uses PKCE: the CLI stores a private verifier locally and exchanges it with the MentionDock API. Enter Google passwords and verification codes only in the browser. The CLI's login output omits the access token and verifier.

API-key login accepts `mentiondock login --api-key "<key>"` for non-interactive use, or a hidden prompt when the key is omitted. Both verify the account before replacing saved credentials and return safe account information. Command arguments can appear in shell history, process listings, or agent execution logs; the hidden prompt keeps manually entered keys out of command arguments.

Purchases open a validated Stripe checkout URL. Payment details are entered on Stripe's page; the CLI does not collect card numbers. The Skill asks for authorization matching the purchase type, including the recurring amount and renewal terms for subscriptions. Data queries consume existing credits.

Browser launching uses fixed operating-system programs with separate arguments. Remote response text is not interpolated into a shell command.

## Checks and their limits

[Snyk's independent report on skills.sh](https://skills.sh/pietrastudio/mentiondock-skill/mentiondock/security/snyk) currently reports two medium-level capability risks: handling third-party social content and offering payment commands. The report recognizes that Stripe performs payment authentication and confirmation outside the agent. Read its current findings before installation; directory availability does not mean a clean scan or certification.

The [dated scan reports](audits/2026-10-21/README.md) include a maintainer-run Cisco AI Skill Scanner check of the exact published Skill: no critical, high, medium, or low findings, with one informational finding about its missing license declaration. The report records the tool version and Skill hash so the scope can be checked.

On 2026-10-21, the published `mentiondock@0.1.2` tarball's integrity was verified, and a maintainer-run `npm audit` of a freshly resolved dependency tree reported **0 known vulnerabilities**.

That result describes npm's known dependency advisories at that time. It is not an independent penetration test, a review of every possible behavior, or a guarantee that vulnerabilities do not exist. Dependency resolutions and advisory databases can change.

The audited CLI tarball is available from the [npm registry](https://registry.npmjs.org/mentiondock/-/mentiondock-0.1.2.tgz). Its SHA-256 is:

```text
be6aa9a0bbd4d5b3eff44c76eb0d69ba06abf051309a287577dd0d793395ff8a
```

A scan of this Skill repository covers its instructions and metadata; it does not automatically cover the separately downloaded CLI, its dependencies, MentionDock's servers, Google, or Stripe. Platform compatibility or publication is not a platform security certification.

## Report a security concern

Use the contact link on [mentiondock.com](https://mentiondock.com) to report a security concern privately. Include the affected version and reproduction steps, with tokens, payment details, and unrelated personal data removed.
