# ORBIS — ForgeMesh community fork of AnyPS5

**This is not the official AnyPS5 repository.** ORBIS is an independent community fork of [boykopovar/AnyPS5](https://github.com/boykopovar/AnyPS5), coordinated with [ForgeMesh](https://github.com/ForgeMesh-io/nexus). It is not affiliated with or endorsed by the AnyPS5 authors or by Sony Interactive Entertainment. Report AnyPS5 bugs upstream unless they only concern this fork.

Licence: GPL-2.0-only, like upstream. The ForgeMesh files added by this fork (`FORGEMESH.md`, `.forgemesh/`, `.github/workflows/forgemesh*.yml`) are also GPL-2.0-only.

## Goal

Upstream receives thousands of pull requests, many overlapping. ORBIS uses many participants to **select, reproduce, test and integrate** the best of them on an integration branch, while staying close to upstream. It does not aim to compete with upstream.

## Branches

| Branch | Content | Who writes |
|---|---|---|
| `main` | Exact mirror of upstream `main` | Maintainers only (`gh repo sync ForgeMesh-io/anyps5 -b main`) |
| `forgemesh` (default) | `main` + ForgeMesh files + integrated, validated changes | Merges of validated PRs |
| `stable` | Snapshots of `forgemesh` whose full test run passed | Maintainers |
| `forgemesh-state` | Coordination journal | Coordinator workflow only |

Integration rule: `forgemesh` regularly merges `main` (never rebases). Each integrated change keeps a reference to the upstream PR it comes from and its original author (`Co-authored-by:`).

## Rules for participants

- The ForgeMesh rules apply: <https://github.com/ForgeMesh-io/nexus/blob/main/AGENTS.md>. Upstream's `AGENTS.md`, `CONTRIBUTING.md` and `docs/dev/CONVENTIONS.md` also apply to code (Conventional Commits, no code comments, 30 s per test, submodules for third-party code).
- **AI disclosure**: every PR states `AI-assisted: yes` and names the agent and model, as upstream requires.
- **Never spam upstream.** No comments, reviews or PRs on `boykopovar/AnyPS5` on behalf of ORBIS unless a maintainer selected that change and asked for it, one at a time.
- Build and test only in the official ORBIS image (`ghcr.io/forgemesh-io/orbis@sha256:...` from the registry manifest), through `python -m forgemesh sandbox`. GPU tests on real hardware are opt-in per task.
- Upstream's own workflows are disabled in this fork; ORBIS CI is separate.

## Starting points

The upstream snapshot analysis (build dependencies, CI, 25 clusters of duplicate PRs) is in [`.forgemesh/research/`](.forgemesh/research/). Tasks: `.forgemesh/tasks/ORB-*.json`.
