# Native ECMAScript Parser Benchmark

Benchmarks for ECMAScript parsers compiled to native binaries (Zig, Rust), measuring raw parsing speed without any JavaScript runtime overhead.

## System

| Property | Value |
|----------|-------|
| OS | macOS 24.6.0 (arm64) |
| CPU | Apple M3 |
| Cores | 8 |
| Memory | 16 GB |

## Parsers

### [Yuku](https://github.com/yuku-toolchain/yuku)

**Language:** Zig

A high-performance & spec-compliant JavaScript/TypeScript compiler written in Zig.

### [Oxc](https://github.com/oxc-project/oxc)

**Language:** Rust

A high-performance JavaScript and TypeScript parser written in Rust.

### [SWC](https://github.com/swc-project/swc)

**Language:** Rust

An extensible Rust-based platform for compiling and bundling JavaScript and TypeScript.

## Benchmarks

### [typescript.js](https://raw.githubusercontent.com/yuku-toolchain/parser-benchmark-files/refs/heads/main/typescript.js)

**File size:** 7.83 MB

![Bar chart comparing native parser speeds for typescript.js](charts/typescript.png)

| Parser | Median | Min | Max |
|--------|--------|-----|-----|
| Oxc | 27.99 ms | 27.32 ms | 33.85 ms |
| Yuku | 28.72 ms | 28.02 ms | 29.69 ms |
| SWC | 55.56 ms | 54.03 ms | 61.89 ms |

### [three.js](https://raw.githubusercontent.com/yuku-toolchain/parser-benchmark-files/refs/heads/main/three.js)

**File size:** 1.96 MB

![Bar chart comparing native parser speeds for three.js](charts/three.png)

| Parser | Median | Min | Max |
|--------|--------|-----|-----|
| Oxc | 7.35 ms | 7.13 ms | 15.41 ms |
| Yuku | 8.21 ms | 8.00 ms | 19.35 ms |
| SWC | 12.71 ms | 12.53 ms | 13.26 ms |

### [react.js](https://raw.githubusercontent.com/yuku-toolchain/parser-benchmark-files/refs/heads/main/react.js)

**File size:** 0.07 MB

![Bar chart comparing native parser speeds for react.js](charts/react.png)

| Parser | Median | Min | Max |
|--------|--------|-----|-----|
| Oxc | 1.42 ms | 1.25 ms | 3.44 ms |
| Yuku | 1.56 ms | 1.41 ms | 9.06 ms |
| SWC | 1.82 ms | 1.63 ms | 11.73 ms |

## Semantic

The ECMAScript specification defines a set of early errors that conformant implementations must report before execution. Some of these are detectable during parsing from local context alone, like `return` outside a function, `yield` outside a generator, invalid destructuring, etc. Others require knowledge of the program's scope structure and bindings, such as redeclarations, unresolved exports, private fields used outside their class, etc.

Parsers handle this differently: SWC checks some scope-dependent errors during parsing itself, while Yuku and Oxc defer them entirely to a separate semantic analysis pass. This keeps parsing fast and lets each consumer opt in only to the work it actually needs. A formatter, for example, only needs the AST and should not pay the cost of scope resolution.

The benchmarks below measure parsing followed by this additional pass, which builds a scope tree and symbol table, resolves identifier references to their declarations, and reports the remaining early errors. Together, parsing and semantic analysis cover the full set of early errors required by the specification.

### [typescript.js](https://raw.githubusercontent.com/yuku-toolchain/parser-benchmark-files/refs/heads/main/typescript.js)

![Bar chart comparing parser speeds with semantic analysis for typescript.js](charts/typescript_semantic.png)

| Parser | Median | Min | Max |
|--------|--------|-----|-----|
| Yuku + Semantic | 47.12 ms | 46.11 ms | 94.11 ms |
| Oxc + Semantic | 64.48 ms | 62.72 ms | 139.23 ms |

### [three.js](https://raw.githubusercontent.com/yuku-toolchain/parser-benchmark-files/refs/heads/main/three.js)

![Bar chart comparing parser speeds with semantic analysis for three.js](charts/three_semantic.png)

| Parser | Median | Min | Max |
|--------|--------|-----|-----|
| Yuku + Semantic | 11.93 ms | 11.68 ms | 24.42 ms |
| Oxc + Semantic | 13.78 ms | 13.49 ms | 23.87 ms |

### [react.js](https://raw.githubusercontent.com/yuku-toolchain/parser-benchmark-files/refs/heads/main/react.js)

![Bar chart comparing parser speeds with semantic analysis for react.js](charts/react_semantic.png)

| Parser | Median | Min | Max |
|--------|--------|-----|-----|
| Yuku + Semantic | 1.74 ms | 1.59 ms | 36.62 ms |
| Oxc + Semantic | 1.76 ms | 1.62 ms | 36.29 ms |

## Run Benchmarks

### Prerequisites

- [Bun](https://bun.sh/) - JavaScript runtime and package manager
- [Rust](https://www.rust-lang.org/tools/install) - For building Rust-based parsers
- [Zig](https://ziglang.org/download/) - For building Zig-based parsers (requires nightly/development version)
- [Hyperfine](https://github.com/sharkdp/hyperfine) - Command-line benchmarking tool

### Steps

1. Clone the repository:

```bash
git clone https://github.com/yuku-toolchain/ecmascript-parser-benchmark-native.git
cd ecmascript-parser-benchmark-native
```

2. Install dependencies:

```bash
bun install
```

3. Run benchmarks:

```bash
bun bench
```

This will build all parsers and run benchmarks on all test files. Results are saved to the `result/` directory.

## Methodology

All parsers are compiled with release optimizations. Source files are embedded at compile time (Zig `@embedFile`, Rust `include_str!`) to eliminate file I/O from measurements. Rust parsers are built with `cargo build --release` using LTO, a single codegen unit, and symbol stripping. Zig parsers are built with `zig build --release=fast`.

Each parser is benchmarked using [Hyperfine](https://github.com/sharkdp/hyperfine) with `--shell=none` to eliminate shell overhead, 30 warmup runs, and a minimum of 200 timed runs. Results use the **median** rather than the mean to provide stable, outlier-resistant measurements. In CI, the CPU frequency governor is set to `performance` mode and processes are pinned to a dedicated core to minimize scheduling noise. Each run measures the time to parse the entire file into an AST and free the allocated memory.