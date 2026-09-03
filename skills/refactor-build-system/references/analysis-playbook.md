# Build Engineering Analysis Playbook

Read only the sections needed for the requested analysis lanes.

## Baseline Inventory

Collect repository evidence before recommending a change:

- Build entry points: wrapper scripts, Makefiles, CMake presets, Bazel files, Gradle/Maven, package scripts, Cargo, Go tooling, Dockerfiles, and developer documentation.
- CI/Pipeline paths: trigger filters, matrices, caches, artifacts, test selection, release jobs, and local-command parity.
- Dependencies: manifests, lockfiles, submodules, vendored code, package-manager configuration, system packages, toolchains, generated code, and runtime libraries.
- Reproducibility and security: version pins, checksums/signatures, download origins, secrets handling, shell interpolation, permissions, SBOM, signing, and provenance.

Record facts and unknowns separately. Static inspection proves configuration shape, not runtime performance.

## Usability

Evidence includes a reproducible contributor command, its actual error/help output, required environment, and the equivalent CI/Pipeline command. Prefer improving an existing entry point over adding another wrapper. Validate syntax, help/error paths, and one narrow successful path when the environment permits.

## Dependencies

Use the build system's native graph and metadata when available: CMake File API or Graphviz output, Bazel `query`/`cquery`/`aquery`, Gradle or Maven dependency reports, `cargo tree`, `go mod graph`, package lockfiles, compiler dependency files, and linker/runtime inspection.

Classify every relevant edge as source, build tool, generated-code, package, system, link, runtime, or hardware-stack dependency. Distinguish direct from transitive and declared from observed. Validate removal or movement with the narrowest build/import/link check that exercises the edge.

## Efficiency

Measure the dimension being changed: clean build, incremental rebuild, test selection, dependency download, cache hit/miss, critical path, or CI/Pipeline wall time. Keep machine, inputs, configuration, and command comparable. Use traces or logs instead of intuition.

Accept a performance change only when output equivalence is checked and the measurement is repeatable enough to support the claim. Avoid changing parallelism blindly; resource contention and memory limits can make higher concurrency slower or unstable.

## Security

Inspect build-time code execution and inputs: untrusted shell interpolation, unsafe temporary paths, writable script or artifact locations, unauthenticated downloads, missing integrity verification, overbroad tokens/permissions, secret leakage, dependency confusion, and mutable release inputs.

For dependency vulnerabilities, capture package identity, resolved version, source, affected range, reachability evidence, and advisory source. For supply-chain improvements, state whether the change affects SBOM completeness, signatures, attestations, or provenance.

## Candidate Record

Use this shape for each finding:

```text
Lane:
Observed evidence:
User or CI impact:
Root cause:
Smallest change:
Files in scope:
Before/after proof:
Security and compatibility impact:
Remaining external gate:
```

Reject candidates without a concrete impact, a bounded file scope, or a reviewable proof path.
