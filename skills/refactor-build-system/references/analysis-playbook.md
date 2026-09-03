# Build Engineering Analysis Playbook

Read only the sections needed for the requested analysis lanes.

## Baseline Inventory

Collect repository evidence before recommending a change:

- Build entry points: Shell wrapper files, Makefiles, Modern CMake files, CMake presets, Bazel files, Maven / Gradle wrappers, Python packaging files, package scripts, Cargo, Go tooling, Dockerfiles, and developer documentation.
- CI/Pipeline paths: trigger filters, matrices, caches, artifacts, test selection, release jobs, and local-command parity.
- Dependencies: manifests, lockfiles, submodules, vendored code, package-manager configuration, system packages, toolchains, generated code, and runtime libraries.
- Reproducibility and security: version pins, checksums/signatures, download origins, secrets handling, shell interpolation, permissions, SBOM, signing, and provenance.

Record facts and unknowns separately. Static inspection proves configuration shape, not runtime performance.

## Usability

Evidence includes a reproducible contributor command, its actual error/help output, required environment, and the equivalent CI/Pipeline command. Prefer improving an existing entry point over adding another wrapper. Validate syntax, help/error paths, and one narrow successful path when the environment permits.

For Shell wrapper work, prefer making the existing facade easier to use: stable subcommands, `--help`, `doctor`, clear default build type, quoted paths, explicit environment variables, and exact parity with CI/Pipeline commands.

For Modern CMake work, prefer `CMakePresets.json` and target-scoped settings over global flags when the repository already supports them. Preserve override precedence for user presets, toolchain files, build type, generator, install prefix, and platform options.

## Dependencies

Use the build system's native graph and metadata when available: CMake File API or Graphviz output, Bazel `query`/`cquery`/`aquery`, Gradle or Maven dependency reports, `cargo tree`, `go mod graph`, package lockfiles, compiler dependency files, and linker/runtime inspection.

Classify every relevant edge as source, build tool, generated-code, package, system, link, runtime, or hardware-stack dependency. Distinguish direct from transitive and declared from observed. Validate removal or movement with the narrowest build/import/link check that exercises the edge.

For Maven / Gradle, capture wrapper version, JDK/toolchain constraints, dependency trees, lock or verification metadata, module graph, cacheable tasks, profiles, and plugin versions before changing dependencies or task wiring.

For Python packaging, inspect `pyproject.toml`, `setup.py`, build backend, wheel/sdist behavior, editable install path, dependency groups/extras, lockfiles, native-extension build inputs, and generated metadata before changing package configuration.

## Efficiency

Measure the dimension being changed: clean build, incremental rebuild, test selection, dependency download, cache hit/miss, critical path, or CI/Pipeline wall time. Keep machine, inputs, configuration, and command comparable. Use traces or logs instead of intuition.

Accept a performance change only when output equivalence is checked and the measurement is repeatable enough to support the claim. Avoid changing parallelism blindly; resource contention and memory limits can make higher concurrency slower or unstable.

For CMAKE_BUILD_PARALLEL_LEVEL or other cache / parallelism changes, document the previous concurrency source, CI worker shape, local override path, memory risk, and fallback behavior. Prefer bounded defaults and user override support over hardcoded maximum parallelism.

For incremental build claims, prove the affected rebuild scope with build-system logs, dependency files, traces, or target-level output. Do not infer incremental behavior from a clean build.

For build performance verification, record the command, machine or runner class, input revision, relevant cache state, timing source, output-equivalence check, and external noise. A single timing can justify investigation; a change claim needs comparable before/after evidence.

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
