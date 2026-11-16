# Changelog

## 0.1.7 — Experimental pre-release

- Offscreen window mode no longer takes keyboard focus on Windows when QFlex opens a window or a tab: each page QFlex opens gets its own inactive window, and a window opened by a page is moved off screen. Popups, link-opened tabs, and dialogs that a page opens can still take the foreground, and under a Linux window manager a small corner of an offscreen window can stay on screen.
- Keyboard input sent right after a page opens is no longer dropped on pages whose first content arrives late or whose response is still streaming, which the 0.1.6 notes listed as a remaining case. Keys sent after the page has presented no longer wait two animation frames each.
- A large page change no longer stalls the calls that follow it. QFlex releases DOM node bindings after each accessibility observation, and headless and offscreen Chrome start with `--disable-platform-accessibility-integration`; on Windows, the worst call after replacing 12,000 elements went from about 7 s to about 40 ms in headless Chrome.
- Broker startup handles a contended profile lock: a short-lived unrelated holder of the lock port no longer fails the start, concurrent starts join the Broker that won, a refusal that can clear is reported as retryable (new Broker exit code 69), and a Broker whose start failed exits within 2 s. On Linux the profile lock is now an abstract Unix socket instead of a loopback port, so an unrelated connection on that port cannot block a start.
- MCP starts a new Broker when a session opens right after the previous Broker stopped, instead of failing on the dead connection, and reports a lost session once.
- On Windows, a managed Chrome close that timed out is evaluated again on the next use, so the profile becomes usable once Chrome has exited without restarting the Broker.
- `check` accepts image evidence copied from a screenshot result as the tool description instructs.

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
