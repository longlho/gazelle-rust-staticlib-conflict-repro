# gazelle Rust staticlib conflict repro

This repository composes `gazelle_py` and `gazelle_ts` into one
`gazelle_binary` to reproduce the second-stage cgo/Rust staticlib link failure.

The baseline pins both language plugins after their public C ABI symbols were
namespaced, but before `gazelle_ts` localizes the non-ABI Rust symbols inside
its `staticlib`.

```sh
bazel build //:gazelle_bin
```

Expected result on Linux: the final cgo link fails because both language plugins
bring a Rust `staticlib` into the same Go binary. Without staticlib symbol
localization, the linker sees duplicate Rust runtime symbols.

`git_override` is not what triggers the bug. The overrides only select exact
intermediate upstream commits for a stable repro:

- `gazelle_py@5b4e319` exports namespaced C ABI symbols:
  `gazelle_py_ie_dispatch` / `gazelle_py_ie_free`.
- `gazelle_ts@e8d2ad5` exports namespaced C ABI symbols:
  `gazelle_ts_ie_dispatch` / `gazelle_ts_ie_free`, but still exposes Rust
  `staticlib` internals globally.

Before that namespacing, both plugins exported generic `ie_dispatch` /
`ie_free` symbols. That earlier collision could accidentally let the final Go
link satisfy both cgo references from one symbol set, masking the Rust runtime
duplicate-symbol problem. Once the C ABI symbols are distinct, the final link
must pull both Rust staticlibs, exposing duplicate `std`/`core`/`alloc` symbols.
