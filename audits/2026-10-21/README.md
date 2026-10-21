# Checks for MentionDock 0.1.2

These checks were run by the maintainer on 2026-10-21. They are reproducible tool results, not a security certification or a manual review by Cisco or npm.

| Check | Scope | Result |
| --- | --- | --- |
| [Cisco AI Skill Scanner 2.3.0](https://github.com/cisco-ai-defense/skill-scanner) | The published `skills/mentiondock/SKILL.md`; static and behavioral analyzers | 0 critical, high, medium, or low findings; 1 informational finding for the missing license declaration |
| npm audit | Freshly resolved production dependencies of `mentiondock@0.1.2` | 0 known vulnerabilities |

Reports: [Cisco result](cisco-skill.json), [npm audit result](npm-audit.json), [dependency snapshot](package-lock.json).

The Cisco report's machine-specific `skill_path` was replaced with the repository-relative path. Findings and other result fields are unchanged. No private source, credentials, or customer data was sent to a scanning service. Cloud and LLM analyzers were not enabled.

Skill SHA-256:

```text
bceeb7f40e1dd0d90feb124a0c567fe4de72b28a3c5ad94355f21248ddd25e02
```

CLI tarball: [registry.npmjs.org/mentiondock/-/mentiondock-0.1.2.tgz](https://registry.npmjs.org/mentiondock/-/mentiondock-0.1.2.tgz)

CLI tarball SHA-256:

```text
be6aa9a0bbd4d5b3eff44c76eb0d69ba06abf051309a287577dd0d793395ff8a
```

To repeat the Skill check after installing the scanner from its official PyPI package:

```sh
uvx --from cisco-ai-skill-scanner==2.3.0 skill-scanner scan skills/mentiondock --use-behavioral --policy balanced --format json
```

To repeat the dependency advisory check, run `npm audit --omit=dev --ignore-scripts` from this directory using its saved lockfile. Advisory results can change over time. The Skill scan does not cover CLI behavior or MentionDock's backend, and the dependency check does not detect every kind of malicious behavior.
