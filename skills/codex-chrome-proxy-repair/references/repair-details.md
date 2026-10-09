# Technical repair notes

## Proxy inheritance

Observed chain: Desktop/app-server → `@oai/cua-repl/bin/cua-repl.mjs` → native `node_repl` → trusted Node worker → browser service. In the reproduced case, parent HTTP proxy settings were absent in the helper, which attempted direct external TCP connections. Correcting the trusted launch environment restored tab enumeration without requiring TUN.

Locate the current installation through process command lines and package exports. Do not hardcode a runtime hash or plugin version. Inspect the launcher's `spawn` options to establish whether it preserves environment values.

Illustrative patch logic, only after the agent has obtained `verifiedHttpProxy` from the user's active configuration:

```js
for (const [upper, lower] of [
  ["HTTP_PROXY", "http_proxy"],
  ["HTTPS_PROXY", "https_proxy"],
]) {
  const value = process.env[upper] ?? process.env[lower] ?? verifiedHttpProxy;
  if (value) {
    process.env[upper] ??= value;
    process.env[lower] ??= value;
  }
}
// Only if supported by the installed Node runtime and appropriate to its usage:
process.env.NODE_USE_ENV_PROXY ??= "1";
```

This is a pattern, not a standalone executable: `verifiedHttpProxy` is deliberately not supplied. Preserve conflicting explicit settings and investigate them instead of silently selecting one. Respect intentional empty values. Preserve existing NO_PROXY/no_proxy; if absent, derive appropriate bypass rules from the environment, including required local services. `ALL_PROXY` support varies by client; setting it alone is not proof Node fetch uses it. Keep credentials out of source and public reports; credential-bearing endpoints need protected configuration.

Check launcher syntax with its bundled Node. A child-process probe should verify actual inheritance and preservation of pre-existing values. Proxy probe success is only transport validation.

## Saved-permission failure after network repair

In the observed runtime, browser service reads global `browser/config.toml` and per-chat `browser/sessions/<session>.toml` through a privileged config bridge. A caught exception can become a generic saved-permissions error. The normal model-facing `nodeRepl.config` being absent is intentional and does not prove the trusted bridge is missing.

Inspect the actual CODEX_HOME and file ancestors with lstat/reparse metadata; do not infer them from shell defaults. Record the requested relative config path and original exception if safe diagnostic logging is necessary. Do not log config contents or grants. Restore only the known logging change or byte-compare against the expected instrumented original before restoration.

### Genuine symlink

If CODEX_HOME, browser, sessions, or a relevant ancestor really is a link, investigate the ownership/layout purpose. Public macOS reports describe replacing linked browser directories with real directories while preserving bytes and permissions. That is not a universal Windows fix. Any authorized layout correction must preserve grants, all files, and rollback; never fabricate approvals or automatically relocate an entire Codex home.

### Windows drive-prefix false positive

A separate case was reproduced on Windows with ordinary CODEX_HOME/browser directories and a junction working directory.

Read-only inspection of the installed native binary showed `reject_symlink_ancestors` accumulating path components and calling `symlink_metadata` after each component, including a bare drive prefix before the root separator. Windows resolves `C:` as the current directory on that drive, whereas `C:\` is the drive root. If the helper cwd is a junction on that drive, checking `C:` can reject an unrelated link before reaching the ordinary config directory.

Required evidence before applying this remedy:

- Captured underlying TOML directory-symlink exception.
- Real config path and relevant ancestors are ordinary directories.
- Actual native-helper cwd is a junction/link.
- In that launch context, `lstatSync(drivePrefix)` reports a link while the full drive root does not.
- Resolving cwd to its physical path keeps the same directory identity and removes the false-positive condition. Inspect inherited per-drive working directories too; another machine may differ.

The tested launcher-side workaround runs before spawning native node_repl:

```js
if (process.platform === "win32") {
  const { realpathSync } = await import("node:fs");
  process.chdir(realpathSync.native(process.cwd()));
}
```

Use this only in the affected browser helper, not as a global shell change. It does not alter permission decisions or config files; it changes the helper's launch path to the same physical workspace. Verify directory identity, the resulting drive-relative lookup, and original page control after an authorized helper reload. It works around the native implementation; it does not patch that implementation or cure actual config symlinks.

## Verified case and limits

On 2026-10-09, Windows Desktop build `26.1002.7124.0` with bundled Node `24.21.0` exhibited both faults sequentially. After proxy inheritance and canonical-cwd changes, the official Chrome interface enumerated tabs, read the previously failing public page, expanded a language menu, and collapsed it again. This proves the tested read/click path in that installation, not universal compatibility. No saved grants were changed, and Desktop/main app-server were not restarted for these repairs. Temporary service diagnostics were removed.

## Public leads to re-check

Issue reports are user evidence, not proof of an official shipped fix. Re-open relevant sources before relying on their current status.

- [Proxy/TUN symptoms, #44364](https://github.com/openai/codex/issues/44364)
- [Chinese proxy diagnosis, #44117 comment](https://github.com/openai/codex/issues/44117#issuecomment-5690737864)
- [Chinese browser proxy write-up](https://youngship.github.io/notes/2026/09/20/codex-browser-proxy/)
- [Generated configuration overwrite, #42454](https://github.com/openai/codex/issues/42454)
- [Actual CODEX_HOME symlink rejection, #35310](https://github.com/openai/codex/issues/35310)
- [Hidden permission snapshot error, #50717](https://github.com/openai/codex/issues/50717)

The last two describe related config-read errors, not a verified upstream patch for the Windows bare-drive-prefix defect.
