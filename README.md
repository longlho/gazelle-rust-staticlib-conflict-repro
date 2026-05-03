# gazelle Rust staticlib conflict repro

This repository composes the current released, unpatched `gazelle_py` and
`gazelle_ts` Gazelle language plugins into one `gazelle_binary`.

```sh
bazel build //:gazelle_bin
```

Expected result on Linux: the final cgo link fails because both language plugins
bring a Rust `staticlib` into the same Go binary. Without staticlib symbol
localization, the linker sees duplicate Rust runtime symbols.
