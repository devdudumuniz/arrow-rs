<!---
  SPDX-License-Identifier: Apache-2.0
  SPDX-FileCopyrightText: See the NOTICE file distributed with this work for additional information regarding copyright ownership
  SPDX-FileContributor: Licensed to the Apache Software Foundation (ASF) under one or more contributor license agreements
-->

# Fork-local Windows recipe (devdudumuniz checkout)

This file exists only in the user's fork. It is not part of upstream
`apache/arrow-rs` and must not be proposed upstream. It describes how to
rebuild a self-sufficient checkout on Windows from the remote, without any
machine-specific tooling.

## What this fork's `main` contains

`main` = upstream `apache/arrow-rs` `main` plus the M09 documentation work
for issue [#10931](https://github.com/apache/arrow-rs/issues/10931)
(`parquet/src/file/properties.rs`, rustdoc only):

- merge of `6a70a897` "docs(parquet): clarify best-effort writer page limits";
- follow-up commit pointing the page-limit getters at the shared section.

Issue #10931 is assigned to another contributor and
[#12322](https://github.com/apache/arrow-rs/pull/12322) addresses it upstream.
Do not open a competing upstream PR from this work. If #12322 is merged,
resolve `properties.rs` in favour of upstream and drop the local doc delta
by a normal merge.

Branches kept for history only:

| Branch | Head | Status |
|---|---|---|
| `contrib/oss-m09-writerproperties-doc-20261117` | `6a70a897` | integrated into `main` |
| `contrib/oss-m09-parquet-batch-doc`, `-doc-20261114`, `-doc-current`, `-doc-refresh` | `d262756b`, `74b89b45`, `5778fc23`, `ac64a9e1` | equivalent rebases (same patch-id); superseded by `main` |
| `contrib/oss-m09-parquet-batch` | `530a58c8` | rejected code experiment (benchmark regression); never merge |

## Prerequisites

- Git for Windows.
- Rust via `rustup` (MSVC host). The toolchain is pinned by
  `rust-toolchain.toml`; `rustup` installs it on first use.
- Visual Studio Build Tools with the "Desktop development with C++" workload.
- Optional: `typos-cli` (`cargo install typos-cli`).

Keep build output outside the source tree, one target directory per
repository, set only for the current shell:

```powershell
$env:CARGO_TARGET_DIR = 'C:\dev\rust-targets\arrow-rs'   # any local, non-synced path
$env:CARGO_INCREMENTAL = '0'
```

## Clone and setup

```powershell
git clone https://github.com/devdudumuniz/arrow-rs.git arrow-rs
cd arrow-rs
git remote add upstream https://github.com/apache/arrow-rs.git
git fetch upstream main
git submodule update --init   # parquet-testing and testing data
```

## Verify (scoped to the crate changed by this fork)

```powershell
cargo fmt --all -- --check
typos --config typos.toml parquet/src/file/properties.rs
cargo clippy -p parquet --all-targets --all-features -j 1 -- -D warnings
$env:RUSTDOCFLAGS = '-D warnings'; cargo doc -p parquet --no-deps -j 1
cargo test -p parquet --lib -j 1
```

## Build and start

This is a library workspace; there is no service to start. "Start" means
running an example end to end against the submodule test data:

```powershell
cargo build -p parquet --all-features -j 1
cargo run -p parquet --features arrow --example read_parquet -j 1
```

## Sync with upstream

```powershell
git fetch upstream main
git merge upstream/main          # normal merge; never force-push main
git push origin main
```
