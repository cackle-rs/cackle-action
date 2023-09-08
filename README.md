# Cackle GitHub action

## Usage

* Create a `cackle.toml` by following the instructions in the main
  [https://github.com/cackle-rs/cackle](cackle repo).

## Example workflow - Validate checked-in Cargo.lock

```yml
name: cackle
on: [push, pull_request]

jobs:
  cackle:
    name: cackle check and test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: dtolnay/rust-toolchain@stable
      - uses: cackle-rs/cackle-action@latest
      - run: cackle check
      - run: cackle cargo test
```

This workflow does the following:

* Checks out your code
* Installs a Rust toolchain
* Installs Cackle
* `cackle check`: Checks your cackle.toml against your checked in Cargo.lock
* `cackle cargo test`: Runs your tests under cackle (remove this if it's not something you want to
  check)

## Example workflow - Validate latest semver-compatible dependencies

```yml
name: cackle-cron
on:
  schedule:
    - cron: '45 20 * * *'

jobs:
  cackle:
    name: cackle check and test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: dtolnay/rust-toolchain@stable
      - uses: cackle-rs/cackle-action@latest
      - run: cargo update
      - run: cackle check
      - run: cackle cargo test
```

This workflow is identical to first one, but instead of running on git pushes and pull requests, it
runs once per day at 20:45, then it does a `cargo update` before running `cackle`. This means that
when there are newer, semver compatible versions of your dependencies, these will be checked instead
of those in your checked in `Cargo.lock`.

If you don't have your `Cargo.lock` checked in, then the `cargo update` is redundant and you should
probably combine the two workflows, since they're doing the same thing.

## Specifying a cackle version

If Cackle makes a major breaking change, it will be gated behind a bump in the `version` field in
`cackle.toml`. However bug fixes won't be gated behind a version bump. As such, there's the
possibility that a new version of Cackle may detect an API usage that was previously missed,
resulting in your workflow suddenly reporting a failure. If you'd like to avoid such unscheduled
failures and upgrade Cackle on your own schedule, simply replace `cackle-rs/cackle-action@latest`
with for example `cackle-rs/cackle-action@0.2.0`.

## License

This software is distributed under the terms of both the MIT license and the Apache License (Version
2.0).

See LICENSE for details.

Unless you explicitly state otherwise, any contribution intentionally submitted for inclusion in
this crate by you, as defined in the Apache-2.0 license, shall be dual licensed as above, without
any additional terms or conditions.
