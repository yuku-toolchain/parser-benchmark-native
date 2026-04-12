# Native ECMAScript Parser Benchmark

Benchmarks for ECMAScript parsers compiled to native binaries (Zig, Rust), measuring raw parsing speed without any JavaScript runtime overhead.

## System

| Property | Value |
|----------|-------|
| OS | macOS 24.6.0 (arm64) |
| CPU | Apple M4 Pro (Virtual) |
| Cores | 6 |
| Memory | 14 GB |

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

| Parser | Median | Min | Max | Peak Memory (RSS) |
|--------|--------|-----|-----|----|
| Oxc | 32.49 ms | 29.99 ms | 48.62 ms | 52.8 MB |
| Yuku | 34.08 ms | 29.91 ms | 46.49 ms | 40.7 MB |
| SWC | 64.88 ms | 59.96 ms | 93.14 ms | 88.9 MB |

### [three.js](https://raw.githubusercontent.com/yuku-toolchain/parser-benchmark-files/refs/heads/main/three.js)

**File size:** 1.96 MB

![Bar chart comparing native parser speeds for three.js](charts/three.png)

| Parser | Median | Min | Max | Peak Memory (RSS) |
|--------|--------|-----|-----|----|
| Oxc | 7.90 ms | 6.61 ms | 11.73 ms | 13.0 MB |
| Yuku | 9.68 ms | 8.22 ms | 10.71 ms | 11.4 MB |
| SWC | 13.71 ms | 11.81 ms | 25.22 ms | 21.3 MB |

### [react.js](https://raw.githubusercontent.com/yuku-toolchain/parser-benchmark-files/refs/heads/main/react.js)

**File size:** 0.07 MB

![Bar chart comparing native parser speeds for react.js](charts/react.png)

| Parser | Median | Min | Max | Peak Memory (RSS) |
|--------|--------|-----|-----|----|
| Yuku | 1.59 ms | 1.26 ms | 2.47 ms | 2.0 MB |
| Oxc | 1.61 ms | 1.19 ms | 3.71 ms | 2.3 MB |
| SWC | 2.21 ms | 1.55 ms | 4.22 ms | 3.1 MB |

## Semantic

The ECMAScript specification defines a set of early errors that conformant implementations must report before execution. Some of these are detectable during parsing from local context alone, like `return` outside a function, `yield` outside a generator, invalid destructuring, etc. Others require knowledge of the program's scope structure and bindings, such as redeclarations, unresolved exports, private fields used outside their class, etc.

Parsers handle this differently: SWC checks some scope-dependent errors during parsing itself, while Yuku and Oxc defer them entirely to a separate semantic analysis pass. This keeps parsing fast and lets each consumer opt in only to the work it actually needs. A formatter, for example, only needs the AST and should not pay the cost of scope resolution.

The benchmarks below measure parsing followed by this additional pass, which builds a scope tree and symbol table, resolves identifier references to their declarations, and reports the remaining early errors. Together, parsing and semantic analysis cover the full set of early errors required by the specification.

### [typescript.js](https://raw.githubusercontent.com/yuku-toolchain/parser-benchmark-files/refs/heads/main/typescript.js)

![Bar chart comparing parser speeds with semantic analysis for typescript.js](charts/typescript_semantic.png)

| Parser | Median | Min | Max | Peak Memory (RSS) |
|--------|--------|-----|-----|----|
| Yuku + Semantic | 53.58 ms | 49.89 ms | 85.73 ms | 88.9 MB |
| Oxc + Semantic | 72.09 ms | 66.67 ms | 128.58 ms | 93.5 MB |

### [three.js](https://raw.githubusercontent.com/yuku-toolchain/parser-benchmark-files/refs/heads/main/three.js)

![Bar chart comparing parser speeds with semantic analysis for three.js](charts/three_semantic.png)

| Parser | Median | Min | Max | Peak Memory (RSS) |
|--------|--------|-----|-----|----|
| Yuku + Semantic | 12.20 ms | 10.76 ms | 14.05 ms | 21.3 MB |
| Oxc + Semantic | 14.76 ms | 12.58 ms | 16.92 ms | 21.5 MB |

### [react.js](https://raw.githubusercontent.com/yuku-toolchain/parser-benchmark-files/refs/heads/main/react.js)

![Bar chart comparing parser speeds with semantic analysis for react.js](charts/react_semantic.png)

| Parser | Median | Min | Max | Peak Memory (RSS) |
|--------|--------|-----|-----|----|
| Yuku + Semantic | 1.97 ms | 1.43 ms | 3.14 ms | 3.1 MB |
| Oxc + Semantic | 2.01 ms | 1.53 ms | 2.82 ms | 3.1 MB |

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