# Changelog

## 0.1.14 — Experimental pre-release

- Installing or updating on macOS no longer fails with `smoke_failed` under the default temporary directory: the staged-version smoke test now keeps its Broker socket path within the macOS limit.
- On macOS, installs of 0.1.10 through 0.1.13 still use their own updater, which cannot stage this release. Install 0.1.14 once with the steps in the installation guide; later releases then update automatically.

## 0.1.13 — Experimental pre-release

- The installation guide explains how the MCP server, Broker, native host, and CLI processes work together, when the Broker starts and exits, and how to restart it after changing configuration.
- The installation guide shows how to create a `qflex` command launcher on Windows, macOS, and Linux, lists the CLI commands for operating an install, and describes manual update activation and its `blocked` reasons.
- The installer's final next step now points to the public installation guide.

## 0.1.12 — Experimental pre-release

- QFlex can bind to an authorized Hermes Desktop browser tab and keep the tab binding through human handoff.
- Targeted actions highlight their target by default in regular browser sessions. Embedded Desktop tabs keep action highlighting disabled, and a disabled setting still turns off the highlight in other sessions.
- The QFlex control cursor stays trackable after page navigation and returns when a new document is ready.

## 0.1.11 — Experimental pre-release

- Tasks now wait at least 250 ms instead of 500 ms after a click or typed text when the page goes quiet at once. A combobox, or a transition or animation that the input started and that is still running, keeps the 500 ms wait.
- When a page keeps loading after an input, a task that declares completion conditions can wait up to two more steps instead of having to act after a single wait.
- Observations are faster on very large pages: resolving element selectors no longer scans the whole document once per element.
- The bundled Public Suffix List data is newer. A credential scope domain can no longer be one of the newly listed public suffixes, such as `run.app`; a stored scope that names one is rejected and needs a registrable domain under it.

## 0.1.10 — Experimental pre-release

- Public release archives now include a managed installer. Run `node qflex-install.mjs` from the extracted runtime to set up the stable `current` path. Install once from the archive if you use an older installation without that layout.
- Managed installations check for newer releases by default, verify the download against the release checksum, and activate a staged version on a later MCP server start when the installation is idle. Use `qflex update status`, `apply`, or `rollback` to inspect, activate, or restore a version. If an existing native-host launcher points directly to a version, run the host's `register` command once from `current` so future starts can activate updates.
- After a switch changes the extension build, QFlex records the automatic reload attempt before contacting the Broker. A confirmed no-effect result allows a later attempt, and a verified reload clears the pending mark. If the result is uncertain, `qflex doctor` points to manual inspection.
- Extension relay cleanup now handles failed tab creation, setup cancellation, and tabs that stop answering during workspace finish. Tabs kept at finish return to Chrome without consuming Broker capacity. If focus or relay ownership is lost after input may have been sent, QFlex reports the uncertainty so the caller can check the page before acting again.

## 0.1.9 — Experimental pre-release

- On Windows, offscreen window mode now runs Chrome on its own desktop, so in an interactive desktop session no Chrome window, popup, link-opened tab, or page dialog appears on screen or takes focus. A person cannot see or reach that browser, so `workspace` kind `handoff` and local `private_input` are refused there with an explicit reason, and `workspace` kind `finish` closes kept tabs instead of holding them for review. When that desktop cannot be used (for example when the Broker runs outside the interactive window station), QFlex logs `chrome.hidden_desktop` with outcome `fallback` and a reason and opens off-screen windows as in 0.1.8. The lock screen, Remote Desktop sessions, and the UAC secure desktop were not tested, and endpoint protection can flag the PowerShell helper that starts Chrome.
- `qflex broker stop` asks the running Broker to shut itself down and reports what happened. The Broker stops only when nothing is live (no request in flight, extension reload, session opening or closing, owned page, unfinished task, uncertain action, workspace, held or review tab, relay lease, or tracked download) and otherwise names the live categories and changes nothing. This gives Windows a graceful stop for a Broker that the MCP server started. Downloads that Chrome's own default handling started in a Broker-launched Chrome are not counted.
- Several agents sharing one Broker no longer fail each other's calls. Commands to Chrome are admitted fairly (at most 64 in flight, 40 for the tabs of one agent, 12 for one tab; the rest wait their turn and each tab's commands keep their call order), and a command that could not be sent in time is reported as not sent and retryable instead of `tab_lost` or `internal`. With four agents opening 20 tabs each at once, all 80 opens succeeded; before, 32 failed with a false `tab_lost` and 16 with `internal`.
- A tab whose page does not answer can always be released. `tab` kind `close`, `workspace` kind `finish`, and the end of a session close such a tab after a short wait even when the page is frozen, without cutting a call that may still be changing the page. Closing 20 frozen tabs took under 3 seconds on Windows and Linux; before, none closed within 30 seconds. A read that timed out on such a page is reported as a retryable `timeout`, and so is a call whose page did not answer before any input was sent.
- When no person can see the browser (headless Chrome, or offscreen mode on Windows), tabs kept at `workspace` kind `finish` are closed instead of holding Broker page capacity for a review nobody can do, and the result says so with `keptClosedReason`; an uncertain detached workspace closes its tabs at expiry for the same reason. The extension relay always counts as visible. Capacity refusals now say which limit was reached and when capacity frees.
- `tab` kind `list` stays responsive while one tab hangs: each tab gets 3 seconds, and an entry read from Chrome's target list instead of the page is marked `stateFromTarget`. A failed `tab` kind `open` says whether a tab was left open (`tabOpened`, `tabCloseUnconfirmed`).
- The window-mode documentation records what was measured in Wayland sessions: Chrome on the Wayland platform cannot place its own windows, so offscreen mode does not move them off screen there; use `headless`, or run the Broker under `xvfb-run` with `WAYLAND_DISPLAY` unset.

## 0.1.8 — Experimental pre-release

- On Linux, a start that follows a Broker which died without confirming shutdown now clears the leftover Chrome ownership marker in the managed profile when it can prove that the previous Broker and Chrome are gone, instead of failing with `profile_busy` until the marker is removed by hand. A Chrome that is still running is never closed or adopted, and any unreadable or ambiguous evidence keeps the refusal. The proof assumes one PID namespace per profile and, for a recreated container, an unchanged host name; macOS still reports this recovery as unsupported.
- The Linux scan for Chrome processes that use a profile reads `/proc` directly, so profile paths with non-ASCII characters are matched.
- The offscreen window documentation records what was measured under nine Linux window manager setups: without a window manager and under fluxbox the window is fully off screen, openbox and xfwm4 keep a small corner on screen, and icewm, i3, metacity, mutter, and kwin keep windows or popups on screen; pages stay visible, painting, and accepting input in every case. It no longer says that macOS places an offscreen window off screen: on a macOS 15 runner the window was placed on screen and maximized.

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
