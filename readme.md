# Rosenpass README

![Nix](https://github.com/rosenpass/rosenpass/actions/workflows/nix.yaml/badge.svg)
![QC](https://github.com/rosenpass/rosenpass/actions/workflows/qc.yaml/badge.svg)
![crates.io](https://img.shields.io/crates/v/rosenpass.svg)
![Libraries.io dependency status for latest release](https://img.shields.io/librariesio/release/cargo/rosenpass)

This repository contains

1. A description of the [Rosenpass protocol](https://github.com/rosenpass/rosenpass/raw/papers-pdf/whitepaper.pdf)
2. The reference implementation of the protocol – the [rosenpass tool](./src)
3. A frontend integrating Rosenpass and WireGuard to create a vpn – the [rp frontend](./rp)
4. [Security analysis](./analysis) of the protocol using proverif

## Getting started

First, [install rosenpass](#getting-rosenpass). Then, check out the help functions of `rp` & `rosenpass`:

```sh
rp help
rosenpass help
```

Follow [quick start instructions](https://rosenpass.eu/#start) to get a VPN up and running.

## Contributing

Contributions are generally welcome. Join our [Matrix Chat](https://matrix.to/#/#rosenpass:matrix.org) if you are looking for guidance on how to contribute or for people to collaborate with.

We also have a – as of now, very minimal – [contributors guide](CONTRIBUTING.md).

## Software architecture

The [rosenpass tool](./src/) is written in Rust and uses liboqs[^liboqs]. The tool establishes a symmetric key and provides it to WireGuard. Since it supplies WireGuard with key through the PSK feature using Rosenpass+WireGuard is cryptographically no less secure than using WireGuard on its own ("hybrid security"). Rosenpass refreshes the symmetric key every two minutes.

As with any application a small risk of critical security issues (such as buffer overflows, remote code execution) exists; the Rosenpass application is written in the Rust programming language which is much less prone to such issues. Rosenpass can also write keys to files instead of supplying them to WireGuard With a bit of scripting the stand alone mode of the implementation can be used to run the application in a Container, VM or on another host. This mode can also be used to integrate tools other than WireGuard with Rosenpass.

The [`rp`](./rp) tool written in Rust makes it easy to create a VPN using WireGuard and Rosenpass.

`rp` is easy to get started with but has a few drawbacks; it runs as root, demanding access to both WireGuard
and Rosenpass private keys, takes control of the interface and works with exactly one interface. If you do not feel confident about running Rosenpass as root, you should use the stand-alone mode to create a more secure setup using containers, jails, or virtual machines.

### Networking & ports

rp allocates two UDP ports; if port N is specified for rosenpass, it will allocate port N+1 for WireGuard.

Like WireGuard, Rosenpass does not enforce any separation between clients and servers.
If you do not specify the `listen` option, Rosenpass and WireGuard will choose random ports; this is _client mode_.
If you do not specify `endpoint`, Rosenpass will not try to connect to the peer and instead wait for connections from peers. This is _server mode_.
You may specify both. Leaving out both is not forbidden but also not very useful.

## Security analysis

<!-- Currently, a symbolic analysis in proverif asserts various properties for the Rosenpass protocol. Further on, a proof of the cryptographic promises based on cryptoverif is in the process of being made. -->

We are working on a cryptographic proof of security, but we already provide a symbolic analysis using proverif as part of the software package. You can run the security analysis using the nix package manager which handles installing the dependencies or you can call the [`./analyze.sh`](https://github.com/rosenpass/rosenpass/blob/main/analyze.sh) script directly. In this case, you need to ensure that `proverif`, `graphviz`, `awk`, and `cpp` are installed on your system.

```sh
   (nix) $ nix build .#proof-proverif --print-build-logs
(manual) $ ./analyze.sh
```

The analysis is implemented according to modern software engineering principles: Using the C preprocessor, we where able to split the analysis into multiple files and uses some meta programming to avoid repetition.
The code uses a variety of optimizations to speed up analysis such as using secret functions to model trusted/malicious setup. We split the model into two separate entry points which can be analyzed in parallel. Each is much faster than both models combined.
A wrapper script provides instant feedback about which queries execute as expected in color: A red cross if a query fails and a green check if it succeeds.

[^liboqs]: <https://openquantumsafe.org/liboqs/>

# Getting Rosenpass

Documentation and installation guides can be found at the [Rosenpass website](https://rosenpass.eu/docs).

Rosenpass is packaged for more and more distributions, maybe also for the distribution of your choice?

[![Packaging status](https://repology.org/badge/vertical-allrepos/rosenpass.svg)](https://repology.org/project/rosenpass/versions)

## Docker Images

Rosenpass is also available as prebuilt Docker images:

- [`ghcr.io/rosenpass/rosenpass`](https://github.com/rosenpass/rosenpass/pkgs/container/rosenpass)
- [`ghcr.io/rosenpass/rp`](https://github.com/rosenpass/rosenpass/pkgs/container/rp)

For details on how to use these images, refer to the [Docker usage guide](docker/USAGE.md).

## Benchmarks

This repository contains facilities for benchmarking both the Rosenpass
protocol code and the implementations of the cryptographic primitives used
by it. The primitives are benchmarked using criterion. For the protocol code
benchmarks we use a library for instrumenting the code such that events are
written to a trace, which is then inspected after a run.

Benchmarks are automatically run on CI. The measurements are visualized in the
[Benchmark Dashboard].

[Benchmark Dashboard]: https://rosenpass.github.io/rosenpass/benchmarks

### Primitive Benchmarks

There are benchmarks for the functions of the traits `Kem`, `Aead` and
`KeyedHash`. They are run for all implementations in the `primitives`
benchmark of `rosenpass-ciphers`. Run the benchmarks and view their results using

```
cargo bench -p rosenpass-ciphers --bench primitives -F bench
```

Note that the `bench` feature enables the inclusion of the libcrux-backed
trait implementations in the module tree, but does not enable them
as default.

### Protocol Benchmarks

The trace that is being written to lives in a new module
`trace_bench` in the util crate. A basic benchmark that
performs some minor statistical analysis of the trace can be run using

```
cargo bench -p rosenpass --bench trace_handshake -F trace_bench
```

This runs the benchmarks and prints the results in machine-readable JSON.

---

# Mirrors

Don't want to use GitHub or only have an IPv6 connection? Rosenpass has set up two mirrors for this:

- [NotABug](https://notabug.org/rosenpass/rosenpass)
- [GitLab](https://gitlab.com/rosenpass/rosenpass/)

# Supported by

Funded through <a href="https://nlnet.nl/">NLNet</a> with financial support for the European Commission's <a href="https://nlnet.nl/assure">NGI Assure</a> program.
