# Changelog

## 0.1.6 — Experimental pre-release

- License change: starting with this release, QFlex is distributed under the QFlex End User License Agreement in `LICENSE` instead of the MIT License. Personal use, internal business use, and AI agent integration for those purposes remain free of charge; redistribution, resale, and offering QFlex functionality to third parties as a service require separate written permission. Copies previously supplied under MIT remain governed by MIT, and third-party components keep their own licenses.
- Screenshots requested without a prior observation now take two page observations instead of three, and the tool guidance names an unscoped `fast-dom` observation as the cheapest basis for a screenshot.
- Keyboard input sent right after a page opens in a new browser window now waits for the page to present a frame; Chrome previously dropped such keys while the action was still reported as executed. A page whose first content appears later than its first frames can still drop a key sent in its first half second.
- Accessibility observations resolve element bindings faster on text-heavy pages by skipping text-node snapshot paths.
- Server instructions now tell agents to prefer an available CLI, API, or dedicated MCP for a task and to use QFlex for browser-specific work.
- Releases are now built, smoke-checked, and published from verified commits by an automated workflow, with a release manifest alongside the archives and `SHA256SUMS`.

## 0.1.5 — Experimental pre-release

- Initial public preview distribution of the experimental QFlex runtime as ZIP and tar.gz archives with SHA256SUMS.
- Archives contain compiled JavaScript and runtime assets; original TypeScript source, source maps, and Git history are not included.
- The MCP browser runtime supports tab navigation, page observation, registered browser actions, deterministic checks, and human handoff for supported site-access challenges.
- Setup requires Node.js 26.7.0 and pnpm 12.11.2; installation downloads production dependencies after extraction.

This release does not establish broad website compatibility, connected-agent acceptance, or support for specific operating systems.
