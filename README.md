# QFlex

QFlex is an experimental browser for AI agents, exposed through a local stdio Model Context Protocol (MCP) server. An MCP client can open and navigate session-owned tabs, inspect pages as structured targets, text, or screenshots, perform registered browser actions, and verify observable results with deterministic checks.

QFlex runs browser actions within the calling agent's requested scope. Its current workflows support human handoff for sign-in or supported site-access challenges, followed by a fresh page observation before work continues.

## Status

QFlex 0.1.5 is an experimental pre-release provided for preview and evaluation. Website coverage, connected-agent acceptance, and support on specific operating systems have not been established.

## Download and install

Download the runtime archives and checksum file from [GitHub Releases](https://github.com/whackur/qflex/releases). Releases provide a ZIP archive, a tar.gz archive, and SHA256SUMS. The archives contain compiled JavaScript and runtime assets. Installation requires Node.js 26.7.0 and pnpm 12.11.2; setup downloads production dependencies after extraction.

Follow the [installation guide](docs/installation.md) for setup steps.

## Features

- Navigate tabs owned by a QFlex session and inspect their pages as structured targets, text, or screenshots.
- Perform registered browser actions against observed pages and check observable conditions with deterministic checks.
- Hand a session to a person for sign-in or a supported site-access challenge, then continue from a fresh observation.
- Preserve uncertain action outcomes for inspection instead of replaying them automatically.

## Community feedback

Use the [bug report form](https://github.com/whackur/qflex/issues/new?template=bug_report.yml) to report a reproducible problem or the [feature request form](https://github.com/whackur/qflex/issues/new?template=feature_request.yml) to propose a change. See [Contributing](CONTRIBUTING.md) for the issue workflow.

## Repository contents

This repository hosts public documentation, issue tracking, and release assets. QFlex runtime source code is maintained separately.

## License

QFlex is distributed under the MIT License; see [LICENSE](LICENSE).
