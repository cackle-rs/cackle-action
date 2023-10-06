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
      - run: cargo acl -n
      - run: cargo acl -n test
```

This workflow does the following:

* Checks out your code
* Installs a Rust toolchain
* Installs Cackle
* `cargo acl -n`: Checks your cackle.toml against your checked in Cargo.lock
* `cargo acl -n test`: Runs your tests under cackle (remove this if it's not something you want to
  check)

## Example workflow to avoid running Cackle too much

Cackle currently always effectively does a debug build from scratch. It also enables debug
information for build scripts, which is normally turned off. This combined with the bit of time it
takes Cackle to actually analyse your object files and binaries means that Cackle will likely be
the slowest part of your CI.

For this reason, you might like to only trigger the cackle workflow when either Cargo.lock or
cackle.toml is changed. You can do this by changing the `on` section of your workflow as follows:

```yml
on:
  push:
    paths:
      - Cargo.lock
      - cackle.toml
  pull_request:
    paths:
      - Cargo.lock
      - cackle.toml
```

Triggering the workflow only when Cargo.lock or cackle.toml is changed means that you won't have to
wait for Cackle to run on most pull requests. This covers the majority of cases when running Cackle
would be necessary. You should only use this however if you're prepared for an occasional failure to
sneak through, since it is possible to have a PR or commit that could cause Cackle to fail without
changing either of these files. For example:

* You could call add unsafe code or use a new API category from your own crate, requiring that you
  grant your own crate this access in order for Cackle to pass.
* Your previous usage of one of your dependencies might have meant that the dependency didn't use
  any filesystem APIs in any reachable code, but now you're using more of that dependency and so
  functions that use filesystem APIs have become reachable. An example would be if you were using
  the `image` crate to decode images in memory, so hadn't needed to grant the image crate the `fs`
  API, but later you added a test to your own crate that used the `image` crate to write an image
  file to a temporary directory.

If you do decide to only trigger your workflow when these two files change, you should probably have
a cron job (see below) that unconditionally runs cackle so that you pick up if any changes sneak
through and need an update to cackle.toml after the fact.

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
      - run: cargo acl -n
      - run: cargo acl -n test
```

This workflow is identical to first one, but instead of running on git pushes and pull requests, it
runs once per day at 20:45, then it does a `cargo update` before running `cargo acl`. This means
that when there are newer, semver compatible versions of your dependencies, these will be checked
instead of those in your checked in `Cargo.lock`.

If you don't have your `Cargo.lock` checked in, then the `cargo update` is redundant and you should
probably combine the two workflows, since they're doing the same thing.

## Specifying a cackle version

If Cackle makes a major breaking change, it will be gated behind a bump in the `version` field in
`cackle.toml`. However bug fixes won't be gated behind a version bump. As such, there's the
possibility that a new version of Cackle may detect an API usage that was previously missed,
resulting in your workflow suddenly reporting a failure. If you'd like to avoid such unscheduled
failures and upgrade Cackle on your own schedule, simply replace `cackle-rs/cackle-action@latest`
with for example `cackle-rs/cackle-action@0.3.0`.

## Caching

If you decide to add caching to your job, there's currently no point caching the `target` directory
because Cackle currently does a `cargo clean` each time it runs (except when using the `test` or
`run` subcommands). Eventually Cackle may be able to remove this `cargo clean`, in which case this
advice will be updated.

## License

This software is distributed under the terms of both the MIT license and the Apache License (Version
2.0).

See LICENSE for details.

Unless you explicitly state otherwise, any contribution intentionally submitted for inclusion in
this crate by you, as defined in the Apache-2.0 license, shall be dual licensed as above, without
any additional terms or conditions.
