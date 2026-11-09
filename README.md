# Codkit application package

This package installs [Codkit](https://github.com/vibevm/codkit) 1.0.0, the
patch-set manager for fork trees, as a portable Vibe user application. It
contains no binaries: each installation downloads one pinned upstream archive
and verifies it against its SHA-256 digest.

| Archive | From | Digest from |
| --- | --- | --- |
| `codkit-1.0.0-<platform>.tar.gz` / `.zip` (offline bundle: Codkit, its project types and every npm dependency, including the native `@resvg/resvg-js` binary) | the [Codkit v1.0.0 release](https://github.com/vibevm/codkit/releases/tag/v1.0.0) | the release's `SHA256SUMS` |

Codkit is a Node.js program and needs Node.js 22.6 or newer (its
`engines.node` is `>=22.6.0`). The package brings no runtime of its own: the
`codkit` launcher is declared `executable = "command:node"`, so it runs
whichever `node` is on PATH when `codkit` starts.

```text
vibe install -g org.vibevm.codkit/codkit
codkit --help
codkit types
```

On a machine with no Node.js, the install still succeeds — the plan says that
`node` is not on PATH — and Vibe's own Node.js package supplies one:

```text
vibe install -g org.vibevm.node/node
```

Nothing binds Codkit to that package: any Node.js 22.6+ on PATH does, from the
distribution, from nvm, or from a corporate image.

The command appears in `opt/bin` of the Vibe settings directory (`~/.vibe/opt/bin`
by default) as a PATH proxy; put that directory on PATH (`vibe self doctor
--fix`), or open a shell with `vibe shell`. Codkit 1.0.0 has no `--version`
flag; `codkit types` prints the installed versions of Codkit's project types.

## Targets

| Installer option | `--platform` / `--arch` | Codkit asset |
| --- | --- | --- |
| `linux-x64` | `linux` / `x86_64` | `codkit-1.0.0-linux-x64.tar.gz` |
| `linux-arm64` | `linux` / `aarch64` | `codkit-1.0.0-linux-arm64.tar.gz` |
| `linux-musl-x64` | `linux-musl` / `x86_64` | `codkit-1.0.0-linux-x64-musl.tar.gz` |
| `macos-x64` | `macos` / `x86_64` | `codkit-1.0.0-darwin-x64.tar.gz` |
| `macos-arm64` | `macos` / `aarch64` | `codkit-1.0.0-darwin-arm64.tar.gz` |
| `windows-x64` | `windows` / `x86_64` | `codkit-1.0.0-win32-x64.zip` |
| `windows-arm64` | `windows` / `aarch64` | `codkit-1.0.0-win32-arm64.zip` |

Vibe picks the option of the host's platform and architecture and asks when it
cannot decide. The options differ only in the archive they fetch: Codkit's
bundle carries a platform-native npm dependency, which is why there is one per
target. A musl system (Alpine and others) is detected as `linux` like any other
Linux, so install there with `vibe install -g org.vibevm.codkit/codkit
--platform linux-musl`.

## Runtime requirements outside the package

- `node` 22.6+ on PATH: the `codkit` launcher runs it. `vibe app plan`
  warns before the download when there is none; the launcher itself says so
  when it is started without one.
- `git` on PATH: Codkit drives Git for every fork-tree operation.
- The Claude Code hook Codkit writes into a fork tree (`hooks/pretool.mjs`) is
  started by Claude Code as `node …` from PATH — the same `node` this package
  already relies on.

## Closed networks

The same package, unedited, installs where GitHub cannot be reached. The
administrator mirrors the registry into the corporate one and places the
archives in the machine's `[artifacts] packages` drop; every archive is still
verified against the digest in `vibe.toml`.

1. Mirror the package (on a connected machine):

   ```text
   vibe registry mirror vibespecs --into <corporate monorepo url> --package org.vibevm.codkit/codkit
   ```

2. Download the archives of the targets you need from the Codkit release above
   and place them in the drop. `vibe artifacts where
   org.vibevm.codkit/codkit` prints, for every installer option, the exact
   file names, digests and destination paths; the layout is

   ```text
   <drop>/org.vibevm.codkit/codkit/1.0.0/codkit-1.0.0-linux-x64.tar.gz
   <drop>/org.vibevm.codkit/codkit/1.0.0/codkit-1.0.0-win32-x64.zip
   …
   ```

3. A site profile names the registry, the drop and the source policy, for
   example

   ```toml
   schema = 1
   name = "example-site"

   [[registry]]
   name = "corp"
   type = "monorepo"
   url = "ssh://git.example.internal/vibe/registry.git"

   [source_policy]
   exclude_hosting = ["github", "gitverse"]

   [artifacts]
   packages = "/srv/vibe-drop"
   ```

   applied with `vibe self apply site.toml`; then `vibe install -g
   org.vibevm.codkit/codkit` reads the package from `corp` and the archive from
   the drop, with no connection to the public hosts. It also works with
   `--offline`.

With a proxy instead of a drop, the archive is downloaded through the
`[proxy]` the site profile declares (`mode = "px"` with a PAC file is
supported).

## Maintaining

`vibe.toml` is **rendered, never edited**. Its renderer is a package of its
own — `org.vibevm.codkit/vibevm-package-generator`, which carries the whole
`org.vibevm.codkit` family — so this repository ships no tooling:

```text
vibe install -g org.vibevm.codkit/vibevm-package-generator
vibevm-generate-package-codkit --codkit 1.0.0 SHA256SUMS SIZES > vibe.toml
```

`SHA256SUMS` is the release's own checksum asset and `SIZES` a file of
`<bytes> <name>` lines for the same archives (`stat -c '%s %n' *`). The
renderer fetches nothing; the rendered header records the generator version
and the SHA-256 of both inputs, and re-rendering from the same inputs is
byte-identical.

To move to a new Codkit release: download every archive, verify it against the
upstream checksum file, check that each archive's top-level directory still
matches `payload_root` and the launcher's argument, render, re-render and
compare, and install on at least Linux and Windows.

## License

This package (its manifest and documents) is UPL-1.0, see `LICENSE.md`.
Codkit is UPL-1.0 (its bundle carries its own `LICENSE.md` and the licenses of
its npm dependencies).
