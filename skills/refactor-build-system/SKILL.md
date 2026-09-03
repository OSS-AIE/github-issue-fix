---
name: refactor-build-system
description: Use when a repository task mentions build-script usability, CMake/CMakePresets/Modern CMake, CMAKE_BUILD_PARALLEL_LEVEL, Maven/Gradle, Python packaging, shell wrappers, incremental builds, cache/parallelism, build/test speed, dependency graphs, reproducibility, CI build failures, SBOM/provenance, or build supply-chain security.
---

# Refactor Build System

## Overview

Turn build-engineering observations into a small, evidence-backed change that preserves outputs and follows the repository's native build and contribution workflow.

## Trigger Hints

Use this skill for mainstream build-framework cleanup and optimization, including:

- Modern CMake: `CMakeLists.txt`, `CMakePresets.json`, toolchain files, `find_package`, `target_link_libraries`, install/export rules, generated files, and `compile_commands.json`.
- CMAKE_BUILD_PARALLEL_LEVEL: parallel build defaults, CI worker sizing, memory-sensitive job limits, and platform-specific overrides.
- Maven / Gradle: dependency reports, wrapper usage, build cache, test selection, profiles, lifecycle phases, and plugin configuration.
- Python packaging: `pyproject.toml`, `setup.py`, wheels, editable installs, dependency groups, build backends, and lockfiles.
- Shell wrapper: `build.sh`, `build.ps1`, Makefile facade, help output, doctor/setup commands, quoting, error handling, and local/CI parity.
- incremental build: rebuild scope, generated-code invalidation, dependency edges, timestamp churn, and test impact selection.
- cache / parallelism: local/remote caches, cache keys, artifact reuse, concurrency, resource contention, and reproducibility.
- build performance verification: clean versus incremental timings, trace/log evidence, output equivalence, and before/after comparison.

## Composition Contract

This skill owns build-system analysis, dependency mapping, efficiency analysis, and build security. A platform skill owns remote operations:

- GitHub issue, PR, review, or Actions work: **REQUIRED SUB-SKILL:** Use `github-issue-fix`.
- GitCode issue, PR, review, or Pipeline work: **REQUIRED SUB-SKILL:** Use `gitcode-issue-fix`.

If a platform skill invoked this skill, return the build analysis to that workflow; do not invoke the platform skill again. Never let this skill bypass repository templates, ownership triage, authentication checks, quality gates, or CI/Pipeline follow-up.

## Core Workflow

1. Preserve local work and read repository instructions, contribution guides, build entry points, CI/Pipeline definitions, dependency manifests, lockfiles, container files, and toolchain configuration.
2. Establish a baseline from repository facts. Record the supported build path, clean and incremental validation paths, dependency sources, cache behavior, and security controls. Do not invent timing or cache data.
3. Analyze only the requested lanes. Read `references/analysis-playbook.md` for their evidence and validation requirements.
4. Write a one-sentence issue contract with the observed problem, affected path, expected outcome, and proof method.
5. Choose the smallest change that satisfies the contract. Separate unrelated usability, dependency, performance, test, CI, and security findings into different issues or PRs.
6. Validate the nearest affected layer first, then the platform quality gate. Compare before/after evidence when the claim is about speed, rebuild scope, cache behavior, or dependency reduction.
7. Hand the contract, evidence, changed files, validation results, and remaining risks back to the owning platform workflow.

## Analysis Lanes

| Lane | Required evidence | Typical safe outcome |
|---|---|---|
| Usability | Actual entry points, options, environment checks, docs/CI parity | Clearer defaults, diagnostics, or one shared entry point |
| Dependencies | Declared and observed build/runtime edges, versions, origins | Explicit edge, pinned source, reduced coupling, SBOM input |
| Efficiency | Reproducible timing, trace, rebuild scope, cache or test-selection evidence | Narrower work, stable cache key, safe parallelism |
| Security | Download verification, command construction, permissions, secrets, provenance | Verified inputs, safer execution, SBOM/provenance coverage |

## Guardrails

- Preserve artifact contents, supported platforms, release behavior, and hardware routing unless the issue explicitly changes them.
- Treat formatter-only churn, speculative rewrites, test weakening, and unmeasured performance claims as out of scope.
- Do not report a vulnerability from version text alone; verify reachability and an authoritative advisory when network access is available.
- If hardware, credentials, or scale prevent full validation, run deterministic static or narrow checks and state the exact remaining gate.

## Output Contract

Return: platform and analysis lanes; baseline evidence; issue contract; root cause; minimal change; before/after validation; security impact; remaining CI, Pipeline, hardware, or permission gates.
