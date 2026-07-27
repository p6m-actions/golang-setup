# Go Setup

![Latest Release](https://img.shields.io/github/v/release/p6m-actions/golang-setup?style=flat-square&label=Latest%20Release&color=blue)

## Description

Installs the Go toolchain with module and build caching, and — when the repository needs it — the
protobuf toolchain that gRPC services compile against.

Two things distinguish it from calling `actions/setup-go` directly:

- **The version comes from `go.mod`.** A Go module already declares its language version, so the
  workflow does not restate it. Pass `go-version` only to override.
- **`protoc` is installed on demand.** A gRPC service imports bindings generated from `proto/`, and
  `protoc` is a standalone binary no runner ships. `install-protoc: auto` (the default) installs it
  and the two Go plugins when a `proto/` directory exists, and skips the cost otherwise.

Pair it with [`golang-build`](https://github.com/p6m-actions/golang-build), which runs the
generation itself.

## Usage

```yaml
- uses: actions/checkout@v4
- uses: p6m-actions/golang-setup@v1
```

## Inputs

| Input | Description | Required | Default |
|---|---|---|---|
| `go-version` | Go version to install (e.g. `1.23`). Empty = read from `go-version-file` | No | `""` |
| `go-version-file` | File to read the Go version from | No | `go.mod` |
| `cache` | Enable module and build caching | No | `true` |
| `cache-dependency-path` | Dependency file(s) the cache key is computed from | No | `go.sum`, `go.mod` |
| `install-protoc` | Install protoc + Go gRPC plugins: `auto` (when `proto/` exists), `true`, `false` | No | `auto` |
| `protoc-gen-go-version` | Version of `protoc-gen-go` to install | No | `v1.36.5` |
| `protoc-gen-go-grpc-version` | Version of `protoc-gen-go-grpc` to install | No | `v1.5.1` |
| `install-dependencies` | Run `go mod download` to pre-warm the module cache | No | `false` |
| `working-directory` | Working directory for go commands | No | `.` |

> Keep `protoc-gen-go-version` in step with the production Dockerfile. Generated bindings that
> disagree with the runtime library are a link error, not a warning.

## Outputs

| Output | Description |
|---|---|
| `go-version` | The installed Go version (e.g. `1.23.5`) |
| `cache-hit` | Whether the module/build cache was hit |
| `protoc-installed` | `true` when the protobuf toolchain was installed |

## Examples

### A repository with no `go.mod` at the root

```yaml
- uses: p6m-actions/golang-setup@v1
  with:
    go-version: "1.23"
    working-directory: services/api
```

### Force the protobuf toolchain on, and pre-warm the module cache

```yaml
- uses: p6m-actions/golang-setup@v1
  with:
    install-protoc: "true"
    install-dependencies: "true"
```

## Cache note

`go.sum` is generated rather than rendered, so a freshly scaffolded project may not have one on its
first CI run. The cache key therefore falls back to `go.mod`, which always exists — otherwise the
first build of every new repository would fail to key its cache.
