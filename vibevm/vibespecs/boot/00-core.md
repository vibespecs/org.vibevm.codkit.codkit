# Codkit application package

This package declares how Vibe installs Codkit, the patch-set manager, as a
portable user application on seven targets (Linux glibc x86_64/aarch64, Linux
musl x86_64, macOS x86_64/aarch64, Windows x86_64/aarch64). It contains no
binaries. Each installer option downloads one archive identified by its exact
URL and SHA-256 — the Codkit offline bundle from the Codkit GitHub release.

Codkit needs Node.js 22.6 or newer and the package ships none: the `codkit`
launcher is declared `executable = "command:node"`, so it runs whichever
`node` is on PATH when it starts. The package declares no dependency on any
other package; a user with no Node.js installs one, for example
`vibe install -g org.vibevm.node/node`. That spelling needs Vibe 1.0.12, which
is why `min_vibe_version` is `1.0.12`.

Before changing the package, read `vibe.toml` and `README.md`. **`vibe.toml`
is rendered, never edited**, and its renderer is not in this repository: it is
`org.vibevm.codkit/vibevm-package-generator`, the generator of the whole
`org.vibevm.codkit` family.

```text
vibe install -g org.vibevm.codkit/vibevm-package-generator
vibevm-generate-package-codkit --codkit <version> SHA256SUMS SIZES > vibe.toml
```

The rendered header names the generator version and the SHA-256 of both input
files, and re-rendering from the same inputs is byte-identical — check that
before every publish. When moving to a new Codkit release, download every
archive, verify it against the upstream checksum file, confirm the archive's
top-level directory still matches `payload_root` and the launcher's argument,
render, install on at least Linux and Windows, and keep the public command
name `codkit` stable.
