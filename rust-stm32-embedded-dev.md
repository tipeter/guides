# Rust STM32 Embedded Development on Windows 10

## Rust installation

```
rustup component add rls rust-analysis rust-src rustfmt
cargo install sccache racer cargo-update cargo-make rustsym
```

## Install components for embedded development

```
# target for Cortex-M4F and Cortex-M7F
rustup target add thumbv7em-none-eabihf
rustup component add llvm-tools-preview
cargo install cargo-binutils cargo-generate
```
