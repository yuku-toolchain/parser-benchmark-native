# ECMAScript Native Parser Benchmark

Benchmark ECMAScript parsers implemented in native languages.

## System

| Property | Value |
|----------|-------|
| OS | Linux 6.14.0-1017-azure (x64) |
| CPU | Intel(R) Xeon(R) Platinum 8370C CPU @ 2.80GHz |
| Cores | 4 |
| Memory | 16 GB |

## Parsers

### [Yuku](https://github.com/arshad-yaseen/yuku)

**Language:** Zig

A high-performance & spec-compliant JavaScript/TypeScript compiler written in Zig.

### [Oxc](https://github.com/oxc-project/oxc)

**Language:** Rust

A high-performance JavaScript and TypeScript parser written in Rust.

### [SWC](https://github.com/swc-project/swc)

**Language:** Rust

An extensible Rust-based platform for compiling and bundling JavaScript and TypeScript.

### [Jam](https://github.com/srijan-paul/jam)

**Language:** Zig

A JavaScript toolchain written in Zig featuring a parser, linter, formatter, printer, and vulnerability scanner.

## Benchmarks

### [TypeScript](https://raw.githubusercontent.com/yuku-toolchain/parser-benchmark-files/refs/heads/main/typescript.js)

The TypeScript compiler source code bundled into a single file.

**File size:** 7.83 MB

![TypeScript Performance](charts/typescript.png)

| Parser | Mean | Min | Max | Peak Memory (RSS) |
|--------|------|-----|-----|----|
| Yuku | 63.17 ms | 62.34 ms | 64.62 ms | 39.8 MB |
| Oxc | 63.53 ms | 62.77 ms | 65.11 ms | 53.0 MB |
| SWC | 133.62 ms | 131.64 ms | 135.91 ms | 89.8 MB |
| Jam | 158.01 ms | 156.90 ms | 159.34 ms | 186.1 MB |

### [Three.js](https://raw.githubusercontent.com/yuku-toolchain/parser-benchmark-files/refs/heads/main/three.js)

A popular 3D graphics library for the web.

**File size:** 1.96 MB

![Three.js Performance](charts/three.png)

| Parser | Mean | Min | Max | Peak Memory (RSS) |
|--------|------|-----|-----|----|
| Oxc | 13.33 ms | 13.11 ms | 14.36 ms | 13.5 MB |
| Yuku | 14.08 ms | 13.89 ms | 15.26 ms | 10.1 MB |
| SWC | 26.77 ms | 26.09 ms | 29.10 ms | 22.6 MB |
| Jam | 32.96 ms | 31.81 ms | 34.76 ms | 39.5 MB |

### [Ant Design](https://raw.githubusercontent.com/yuku-toolchain/parser-benchmark-files/refs/heads/main/antd.js)

A popular React UI component library with enterprise-class design.

**File size:** 5.43 MB

![Ant Design Performance](charts/antd.png)

| Parser | Mean | Min | Max | Peak Memory (RSS) |
|--------|------|-----|-----|----|
| Yuku | 48.65 ms | 47.61 ms | 50.16 ms | 30.4 MB |
| Oxc | 50.91 ms | 50.03 ms | 52.21 ms | 41.1 MB |
| SWC | 103.63 ms | 101.46 ms | 105.20 ms | 67.5 MB |
| Jam | Failed to parse | - | - | - |

## Run Benchmarks

### Prerequisites

- [Bun](https://bun.sh/) - JavaScript runtime and package manager
- [Rust](https://www.rust-lang.org/tools/install) - For building Rust-based parsers
- [Zig](https://ziglang.org/download/) - For building Zig-based parsers (requires nightly/development version)
- [Hyperfine](https://github.com/sharkdp/hyperfine) - Command-line benchmarking tool

### Steps

1. Clone the repository:

```bash
git clone https://github.com/arshad-yaseen/ecmascript-native-parser-benchmark.git
cd ecmascript-native-parser-benchmark
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

### How Benchmarks Are Conducted

1. **Build Phase**: All parsers are compiled with release optimizations. Source files are embedded at compile time (Zig `@embedFile`, Rust `include_str!`) to eliminate file I/O from measurements:
   - Rust parsers: `cargo build --release` with LTO, single codegen unit, and symbol stripping
   - Zig parsers: `zig build --release=fast`

2. **Benchmark Phase**: Each parser is benchmarked using [Hyperfine](https://github.com/sharkdp/hyperfine):
   - 100 warmup runs to ensure stable measurements
   - Multiple timed runs for statistical accuracy
   - Results exported to JSON for analysis

3. **Measurement**: Each benchmark measures the total time to:
   - Parse the entire file into an AST (source is embedded at compile time, no file I/O)
   - Clean up allocated memory

### Test Files

The benchmark uses real-world JavaScript files from popular open-source projects to ensure results reflect practical performance characteristics.