---
name: codex-chrome-proxy-repair
description: Diagnose and repair Codex Desktop Chrome/CUA connection failures on Windows, including missing proxy inheritance and saved-browser-permission failures caused by junction working directories. Discover the user's actual proxy and verify real page control.
---

# Codex Chrome connection repair

Use the actual installed runtime and official browser tools. A listed browser or a successful HTTP probe does not prove page control. Preserve authentication, permission policies, unrelated configuration, and existing proxy choices. Respect the user's restart restrictions; never infer permission to restart Desktop or the main app-server from permission to reload a browser helper.

## 1. Establish the failing layer

Record OS, Desktop/core/helper versions, actual executable paths, process ancestry, and the exact first failing operation. Do not assume cached plugin code is the loaded runtime. Resolve the installed package exports and command line.

Use the browser tool's current documentation to enumerate browsers and tabs, bind one relevant public page, and read its accessibility/DOM state. Browser IDs may change after a kernel reset. Match the browser family and extension instance again when necessary.

| Evidence | Investigate |
|---|---|
| `nodeRepl.fetch request failed`, timeout, direct TCP stuck in SYN_SENT | Proxy inheritance and transport |
| Tab enumeration succeeds, page read says `saved browser permissions could not be verified` | Saved permission snapshot/config read |
| `Codex TOML directory must not be a symlink` | Actual config path and Windows drive-relative ancestor checks |
| `Transport closed` immediately after a helper reload | Closed MCP connection; reconnect before judging the patch |

Search official issues and fixes using the exact error, excluding private paths and credentials. Match platform, component, and trigger; similar wording alone is not a diagnosis. Read [the technical reference](references/repair-details.md) for the relevant branch.

## 2. Discover the user's effective proxy

Determine which process makes the failing request. Desktop, Chrome, the CUA launcher, native node_repl, and its trusted Node worker can have different environments.

On Windows, inspect these sources locally, in order of relevance:

1. The failing process's effective `HTTP_PROXY`, `HTTPS_PROXY`, lowercase equivalents, `ALL_PROXY`, `NO_PROXY`, and `NODE_USE_ENV_PROXY`. The shell environment is only a candidate, not proof of inheritance.
2. The same keys in the working parent process, plus user/machine environment settings. Inspect only selected fields; never dump an entire environment containing credentials.
3. `HKCU:\Software\Microsoft\Windows\CurrentVersion\Internet Settings`: `ProxyEnable`, `ProxyServer`, `AutoConfigURL`, `ProxyOverride`. An enabled static proxy is a candidate. A PAC URL is not itself an HTTP proxy endpoint; determine its effective rule for the actual destination using the user's supported client/system facilities.
4. The running VPN/proxy client's own configuration or UI. Confirm its active HTTP/mixed listener, listening socket, owning process, and current routing mode. Do not guess common ports. SOCKS-only and HTTP CONNECT endpoints are not interchangeable.

Compare direct transport with the discovered explicit proxy against a non-sensitive endpoint actually used by the failing runtime. Bound timeouts. HTTP 401/403 can prove transport reachability but do not prove authentication or browser success. Check Node's installed version and support for environment-proxy handling. TUN can affect routing, but it is not the only solution and must not be enabled automatically.

## 3. Apply the smallest justified change

Before editing external runtime/config files, back up the exact object, record its hash, and check for concurrent changes. Inspect original source. Preserve existing proxy and bypass settings. Do not add secrets/proxy variables to the untrusted worker's environment allowlist.

For proven missing inheritance, pass the verified HTTP(S) proxy into the trusted CUA launch chain and enable the installed Node runtime's supported environment-proxy mechanism. Prefer supported configuration where effective. Generated `.mcp.json` files can be overwritten, and an already running helper does not inherit later edits. A targeted launcher patch is a local workaround that updates may replace; do not describe it as an upstream fix.

For permission errors, do not force approval, edit saved grants, disable checks, or access the page through an alternate channel. Capture the underlying config-read exception with minimal temporary diagnostic logging if needed; leave the exception and decision unchanged, then remove the logging. A genuine configuration symlink and a drive-prefix false positive require different remedies.

Present the concrete change when authorization is missing. Reuse existing authorization. Reload only the identified helper when permitted, verifying PID, executable, parent, creation time, and association with the current chat immediately beforehand. A JS kernel reset reloads JavaScript but does not replace the native helper's process environment/cwd. Never kill all Node/Codex processes. If the current MCP transport stays closed after reload, report the blocker and resume on the host's next reconnection; do not claim verification or loop identical retries.

## 4. Verify and deliver

Through the original official Chrome interface:

1. Enumerate actual tabs without inventory errors.
2. Read the same previously failing page and confirm meaningful body content.
3. Perform a harmless reversible interaction, such as expanding and collapsing a menu; observe both changes and restore the starting state. Do not submit a form or alter business data for a test.

Verify the active runtime loaded the patch, temporary diagnostics were removed, unrelated settings were preserved, and prohibited main processes were not restarted. Stop after relevant checks pass; no stress retries are needed.

Report which layers passed, what remains unverified, rollback location, and update-overwrite risk. Do not promise all sites or future versions will work. Public outputs must exclude the user's proxy endpoint, IP, credentials, username, private paths, session IDs, browsing inventory, runtime dumps, and raw local diagnostic artifacts.
