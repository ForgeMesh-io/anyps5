# Upstream research: boykopovar/AnyPS5

Snapshot taken 2026-11-25 for the ORBIS swarm (ForgeMesh). Everything was read through the GitHub REST API (`gh api`). Nothing was cloned or forked, nothing was posted upstream, and no binaries were downloaded or run. Part A lists facts with their source. Part B holds recommendations. README, issue and PR text are the authors' own claims. They are reported here as written and were not verified.

`R` below stands for `repos/boykopovar/AnyPS5`.

---

## Part A: Facts

### 1. Project facts

| Item | Value | Evidence |
|---|---|---|
| Description | "Tool for automatic PS5 executables porting to Linux and Windows" | `R` |
| Nature | A relinker, not an emulator. It converts the PS5 ELF into a native ELF, PE or Mach-O and reimplements the system prx libraries in `core/libs/prx`. The shader recompiler turns GCN/RDNA code into SPIR-V (through glslang). | README, `docs/dev/ARCHITECTURE.md`, tree |
| Licence | GPL-2.0 ("GNU General Public License version 2 only") | `R` (license), README, `LICENSE` |
| Created / last push | 2026-08-03 / 2026-11-25 | `R` |
| Popularity | 61,391 stars, 5,961 forks | `R` |
| Default branch | `main` | `R` |
| Languages (bytes) | C++ 96.99M, Python 6.71M, CMake 2.81M, JavaScript 237k, C 164k, plus GLSL, CSS, Assembly, HTML and Shell | `R/languages` |
| Build system | CMake (`cmake_minimum_required(VERSION 3.20)`; BUILD.md says ≥3.22.1 for the full build and ≥3.24 on macOS) with Ninja, C++20 (`CMAKE_CXX_STANDARD 20`) | `CMakeLists.txt`, `docs/dev/BUILD.md` |
| Compilers | Linux: GCC/G++. Windows: MinGW-w64 GCC (WinLibs 15.2.0 or MSYS2 UCRT64 16.2.0-6; lld is not usable). macOS: AppleClang, relinker only. | BUILD.md, `build.yml` |
| Target OSes | Runtime libraries for Linux and Windows. macOS gets relinker-only builds plus MoltenVK work (issue #1 "macOS support" is open). A web converter (Emscripten) is published on GitHub Pages. | README, BUILD.md, `build.yml`, `release.yml` |
| Graphics backend | Vulkan. A device must support Vulkan 1.1 with graphics, compute and present queues, plus required extensions and features: `VK_KHR_8bit_storage`, `VK_KHR_buffer_device_address`, `VK_KHR_shader_float_controls`, `shaderInt64`, `textureCompressionBC`, and others. Vulkan-Headers must be ≥1.4.330 and the validation layer ≥1.4.335. | `docs/user/USAGE.md` §GPU selection (lines 577, 604), BUILD.md |
| Submodules | 39 in `.gitmodules`. FidelityFX-FSR is vendored as files rather than a submodule (`3rdparty/FidelityFX-FSR/license.txt`). | `.gitmodules`, tree |
| Tests | CTest. `core/libs/tests` has 1,946 tree entries and the per-library tests sit in `*/tests`. BUILD.md mentions "about 950" test executables in a full Windows build. Python tests are in `tools/tests`. Each test must finish within 30 s. GPU tests run on Mesa lavapipe in CI. | tree, BUILD.md line 76, CONTRIBUTING.md |
| Release process | Pushing a `v*` tag runs `release.yml`: Linux on `ubuntu-24.04`, Windows on `windows-latest`, and a web build. It creates or refreshes a **draft** release with `gh release create --draft`, checks the asset list, then publishes it. | `.github/workflows/release.yml` |

**Submodule licences.** These come from `repos/<owner>/<repo>/license` on the GitHub copy, not from the pinned SHA. NOASSERTION means GitHub could not classify the licence. It is not a finding.

| SPDX from API | Submodules |
|---|---|
| Zlib | SDL |
| MIT | VulkanMemoryAllocator, ext-LibAtrac9, mimalloc, nghttp2, quickjs-ng |
| MIT-0 | ZArchive |
| BSD-2-Clause | jxrlib, libwpe |
| BSD-3-Clause | libfvad, ogg, vorbis |
| Apache-2.0 | SPIRV-Tools |
| LGPL-2.1 | ext-ffmpeg-core (KytyPS5), libusb-cmake |
| NOASSERTION | Vulkan-Headers, SPIRV-Headers, glslang, opus, freetype, harfbuzz, stb, tracy, libjpeg-turbo, mbedtls, zstd, vgmstream, zlib, sqlite, fontconfig, curl, libxml2, libpsl, icu, libpng |
| 404 (API) | libressl/openbsd |
| Not on GitHub (not checked) | pixman, cairo (gitlab.freedesktop.org), libyuv (chromium.googlesource.com) |

**CI workflows** (`.github/workflows`, 26 files): area-labels, bug-report-tools, build, conflict-label-pr, conflict-label, conventions, emscripten-libraries, fix-label, input-mapper, labels, pages-progress, pages, pr-intake, pr-labels, pr-overlap, pr-ready, pr-unchanged, progress-comment, progress-report, release, run-report-schema, sanitizers, stale-conflicts, web-release-tools, web-tests, windows-msys2.
- `build.yml` runs on push, pull_request and workflow_dispatch (input `pr` takes up to 40 PR numbers built together). Relinker builds run on `ubuntu-latest`, `windows-latest` and `macos-latest` (with an optional x86_64 Intel-mac build). The full Linux job runs on `[self-hosted, linux, x64, anyps5]` and is skipped on forks (`!github.event.repository.fork`). The full Windows job runs on `windows-latest` with a lavapipe ICD from mesa-dist-win. Configure uses `-G Ninja -DCMAKE_BUILD_TYPE=Release -DBUILD_TESTING=ON` with gcc/g++ and ccache. Tests run with `ANYPS5_REQUIRE_VULKAN=1 GALLIUM_OVERRIDE_CPU_CAPS=avx`. `tools/select_tests.py` can limit the build to changed inputs. The workflow installs the `wild` linker 0.10.0 with its sha256 pinned.
- `sanitizers.yml` runs nightly at 03:41 UTC: ASan and UBSan on `ubuntu-latest`. Known failures are listed in `.github/sanitizer-known-failures.txt`; for example, ASan maps the guest arena above 1 TiB (#16813).
- `windows-msys2.yml` runs nightly at 08:23 UTC.
- `dependabot.yml` updates GitHub Actions.

### 2. Contribution rules

Sources: `CONTRIBUTING.md`, `AGENTS.md`, `.github/pull_request_template.md`, `docs/dev/CONVENTIONS.md`.

- Commit messages follow Conventional Commits. The `Conventions` check (`tools/check_conventions.py`) runs on every PR.
- No code comments, except debt markers, `#endif` and namespace closers, and upstream licence notices. Classes and public methods use PascalCase; private members use camelCase.
- Unimplemented exports call `NotImplemented_nid_no_patch(__func__)`. Implement general behaviour, not per-title hacks.
- New third-party code must be a submodule built from source.
- Each test must finish within 30 s. `build.yml` warns about slower tests ("tests must finish in 30 s (CONTRIBUTING.md)").
- Debt entries go into fragments under `docs/dev/debt/changes/`.
- GPU changes (AGC, shader recompiler, VideoOut) need validation-layer runs on main and on the change, with no new `VUID-` errors. The PR must list the vendor, model, driver and OS of every GPU tested. Shader semantics must cite hardware measurements or ISA/LLVM/ACO/Mesa sources.
- Performance PRs need `tools/trace_digest.py` summaries for main and for the change. Without them the PR gets the `perf numbers missing` label.
- Overlap checks: run `tools/pr_overlap.py --names` or `--branch`.
- Open-PR limit per author: 1 if the author has no merged PR, 20 once at least one is merged, and unlimited from 20 merged. PRs over the limit are closed automatically (label `open PR limit`).
- Full CI for a PR runs only when a collaborator starts it: `gh workflow run build.yml --field pr=...`.
- Labels: `stale` after 7 days with conflicts (the PR is not closed); `already in main`; `review queue` every hour on the 30 oldest ready PRs, with `unblocks title` PRs first.
- PR template fields: What, Tested, Depends on, Fixes, Unblocks, GPUs tested, AI-assisted.
- **AI policy, quoted exactly:** "Say whether the change was written with AI assistance. The author of the pull request is responsible for every line of it." The template line is `AI-assisted: <!-- yes / no -->`.
  - Measured in the open PR bodies (2,666 bodies fetched): `yes` 2,616, `no` 17, field missing 32, other 1.

### 3. Open PR landscape

Search counts: `search/issues?q=repo:boykopovar/AnyPS5+...`. List data: `R/pulls?state=open&per_page=100` paginated, 2,534 PRs captured.

| Metric | Value |
|---|---|
| Open PRs (search) | 2,666 (list pagination returned 2,534, because the set changes during paging) |
| Open drafts | 125 (search); 107 in the list |
| Merged PRs, all time | 16,765 |
| Closed without merge | 5,190 |
| Open / closed issues | 325 / 1,439 |
| Distinct open-PR authors | 494 |
| Number range | #862 to #27205 |
| Age (created) | <7 days: 1,455 · 7–14 days: 609 · 14–30 days: 337 · >30 days: 133 |
| Not updated since 2026-11-18 | 20 (search) |
| Labels on open PRs | docs 1,466 · prx 1,070 · build 874 · agc 698 · **needs rebase 523** · shader 268 · libc 224 · libkernel 204 · perf 120 · relinker 107 · unblocks title 104 (search: 108) · fix error 95 · perf numbers missing 53 · review queue 30 · ci 16 · already in main 5 · stale 1 |
| Title type | fix 1,128 · feat 778 · test 244 · refactor 227 · perf 121 · docs 23 · build 6 · ci 4 · revert 1 |
| Top scopes | agc 438 · libSceNKWebKit 218 · shader 202 · libc 180 · libkernel 173 · tools 126 · relinker 97 · libSceLibreSsl3 84 · libSceIcu 76 |
| Bot PRs | 1: #25282 (Dependabot, "ci: bump the actions group across 1 directory with 2 updates") |
| Stale | 1: #9138 "Bluetooth hid stubs" (`stale`, `needs rebase`) |
| Already in main | #7704, #22162, #22468, #22595, #23060 |
| Body references | 373 PR bodies contain `Depends on #N` |
| GPUs tested | 645 of 2,559 parsed bodies have a value other than n/a. Mentions: windows 439, nvidia 404, amd 274, linux 204, radv 100, lavapipe 76, llvmpipe 74, intel 60, apple 55, moltenvk 53 |

**Top open-PR authors:** sschandra2006 212, slopgobblercobbler 115, gurtgg 82, JJenkx 68, domx4q 65, milescoviello 57, MarcZX10 52, cookie42073-cmd 47.

**Duplicate clusters** (full list in `pr-clusters.json`):
- One mass series of 212 PRs by sschandra2006. 130 share the title "drop redundant errno saves around guest copies", and 200 were opened on 2026-11-25. The sampled PRs are 1 file and 3 deleted lines each (#26941, #27036). The author has 35 merged PRs.
- Several PRs reference the same issue in `Fixes #`: #15234 → 15242/21451/26698; #26881 → 23014/27123; #22509 → 22510/26696; #25225 → 25226/25279; #23485 → 23486/24987; #11977 → 12316/22382; #14439 → 14944/19852; #23480 → 23787/26690; #15742 → 20580/26221.
- Identical titles from different authors: 7256/24083, 5420/26391, 6192/14262, 17661/22996, and four libSceLibreSsl3 pairs (16589/17843, 16981/19125, 17621/19044, 17626/21684).
- Many PR bodies claim to unblock the same title: PPSA01317 (7 PRs), PPSA26344 (6), PPSA99169 (5), CUSA00900 (5), PPSA02015 (5), CUSA00223 (4).

**Sample of 30 PRs** (`R/pulls/N`, `R/pulls/N/files`, numbers spread evenly across the range): most change 1–9 files, one prx library plus its tests and a debt fragment. Outliers: #14829 (+988/−984) and #26064 (23 test files). Mergeable state: clean 20, unstable 6, dirty 4.

### 4. Branches, tags, releases, activity

- Branches (`R/branches`, 12): `main`, `ci/buffer-spans-tests-timeout`, `ci/release-headers-and-unpatched`, `ci/release-hour-timeout`, `ci/update-ci-release-limit-120`, `dependabot/github_actions/actions-136abf1421`, `feat/nid-patcher-exec`, `fix/agc-replay-dma-copy`, `fix/exception-delivery-race`, `fix/web-archive-test-expectations`, `perf/skip-patched-result-memo`, `refactor/no-3rdparty-in-public-headers`. Contributors work from forks.
- Releases (`R/releases`): v0.1.0 2026-09-28 (8 assets), v0.1.1 2026-09-28 (8), v0.1.2 2026-10-14 (9), v0.1.3 2026-10-20 (10), v0.1.4 2026-10-30 (20), **v0.1.5 2026-11-11 (25)**. None are prereleases. Gaps between releases range from 0 to 16 days, about 10 days on average since v0.1.1.
- Tags (`R/tags`): latest is `v0.1.6` → commit 5e9ad5f5, committed 2026-11-24T14:01:49Z. No published release for v0.1.6 appeared in `R/releases`. A draft may exist (drafts are not visible without push access), or it may be unknown.
- Commits on `main` (`R/commits?sha=main&since=...&per_page=1`, count from the last-page link), merge commits included:
  - since 2026-10-26 (30 days): **31,047**
  - since 2026-11-18 (7 days): 5,716
  - since 2026-11-24: 790
  - Latest 5 commits are merges of PRs #26989, #27002, #27001, #26999, #26997, all by oneandonlydean on 2026-11-25.
- Top contributors by commits (`R/contributors`): oneandonlydean 12,886, adriadam10 5,502, MarcZX10 3,064, boykopovar 1,276, EagleBearerYS 1,123.

### 5. Linux build dependencies

See `build-deps.json`. In summary:
- Packages for Ubuntu 24.04 (from `release.yml`, `sanitizers.yml` and BUILD.md): gcc/g++, cmake, ninja-build, ccache, git, python3, libx11-dev, libxext-dev, libxrandr-dev, libwayland-dev, libxkbcommon-dev, libegl-dev, libdecor-0-dev, libpipewire-0.3-dev, libpulse-dev, libasound2-dev, libvulkan1, mesa-vulkan-drivers, locales.
- Commands: `git submodule update --init --recursive`; `cmake -S . -B build -G Ninja -DCMAKE_BUILD_TYPE=Release -DBUILD_TESTING=ON`; `cmake --build build`; `ctest --test-dir build --output-on-failure`.
- Configure downloads FFmpeg binaries unless `-DFFMPEG_PREBUILT_DIR` is set (BUILD.md line 77).
- Write-watch tests need host kernel ≥6.7. Shared-memory import uses `/dev/udmabuf` (USAGE.md line 379), and without access four native depth tests fail instead of skipping (#23709).

### 6. Hardware concerns

GPU vendors that appear in issues and PRs: AMD (RADV and AMDVLK/Windows), NVIDIA (proprietary, Windows and Linux), Intel (ANV, Arc on Windows, iGPUs), Apple (MoltenVK and KosmicKrisp), and software renderers (lavapipe/llvmpipe). Debt fragment names point to RDNA targets gfx1010, gfx1013, gfx1030 and gfx1036. Search totals for open and closed issues: `radv` 249, `llvmpipe OR lavapipe` 208, `nvidia in:title` 74, `intel arc` 43, `vega` 38, `moltenvk` 27, `VK_ERROR_DEVICE_LOST` 26.

**AMD Polaris / GCN / pre-RDNA2:**
- #9759 (open): agc_driver_sample_order_recipe_depth_locations fails on RADV Polaris
- #15316 (open): agc_driver_f32_fma_denormal_mode2 fails on RADV before RDNA2 ("Mesa splits GLSL Fma, even with NoContraction")
- #9781 (closed): agc_idle_publish_wait fails on RADV Polaris
- #3882 (open): AMD Vega iGPU (GCN 5, Windows), 35 agc_driver tests fail
- #26642, #26623 (open): AMD Vega, Windows driver 23.19, depth sample import and strided vertex fetch
- #18480 (open): agc_driver_recorded_draw_staging crashes in amdvlk64.dll (Vega, Windows)
- #14380 (open): BGRA sRGB DCC clears swap red and blue on AMD's Windows driver

**Other AMD:**
- #13538 (open): Ratchet & Clank compute shader becomes 2.5M-word SPIR-V that RADV struggles with
- #23709 (open): native depth tests fail without /dev/udmabuf (RADV)
- #7138 (open): perf regression on RADV REMBRANDT
- #22511 (closed): pixel interlock on RDNA3
- #487 (closed): wave32 pixel shaders on RDNA4
- #3358, #15958 (closed): Radeon 780M on Windows
- #23629 (open): gfx1036 iGPU
- #25366: RX 7900 XTX sweep, 0 VUIDs, 3 AGC failures
- #25612 (closed): RX 9070 RDNA4 sweep
- GAME_TESTING.md says: "A first run that ends in device lost (VK_ERROR_DEVICE_LOST) or a GPU wait timeout while large pipelines are still being built, seen on AMD, often is the cache and not a regression."

**NVIDIA:**
- #2866: LTO relink loses the device with a depth attachment
- #5951, #10634, #6374, #23927: device lost or GPU hang in titles on Windows/NVIDIA
- #10758: LDS tree reduction
- #23262: f16 NaN results differ between NVIDIA and AMD
- #23708: RTX 2060 retile test is flaky
- #19798 (closed): GTX 1070
- USAGE.md line 594: NVIDIA Windows driver 610.47 crashes in `nvgpucomp64.dll`; 617.42 is fine

**Intel:**
- #15371, #15372, #15373: Arc A750 on Windows
- #10126, #10182: Arc fp16 and D32 behaviour
- #15817: Arrow Lake on Linux
- #7013: ANV GPU hang
- #2972: Intel Mesa < 26.2 compile failure
- #6957: UHD 770
- #16883: Intel iGPU stencil

**Software and CI:**
- #24834 (open, 2026-11-21): two AGC tests fail on every main Linux job on the `anyps5-2` runners since 17 Nov and pass on `anyps5-3`
- #11301: titles that bundle PS5_Mesa abort under llvmpipe/WSL
- #25298: Wave32 VCC mask on 8-lane subgroups

**Apple:** #1, #20118, #23480, #7703, #15742, #18785, #24728.

**Process issue:** #4458 (closed) says "Pull requests that change GPU code don't say which GPUs their tests ran on, and a driver-specific regression reached main (#4435)". This is the origin of the GPUs-tested rule.

**Vulkan versions:**
- Runtime needs a Vulkan 1.1 device (USAGE.md line 577).
- Build headers need ≥1.4.330 (BUILD.md line 212).
- Validation layer needs ≥1.4.335.
- Test code references `VK_API_VERSION_1_2` and `VK_API_VERSION_1_3` (`search/code`, test files only).

### Unknown or not checked
- The exact Linux distro and Mesa version of the self-hosted `anyps5` runners.
- The GCC version pinned on Linux.
- How to produce `FFMPEG_PREBUILT_DIR` from source.
- Licences at the pinned submodule SHAs, and of nested submodules.
- Whether a v0.1.6 draft release exists.
- Discussions and "Game bring-up" compatibility data (the Discussions API was not queried).
- Bodies of most issues: only titles were read.

---

## Part B: Recommendations for ORBIS

1. **Never write upstream.** `build.yml` skips its full Linux job on forks and needs a collaborator to dispatch PR builds. ORBIS should therefore run its own container CI (`initial-tasks.json` ORB-001/002) and never trigger upstream workflows.
2. **Throughput is the main constraint.** Upstream has about 1,000 commits a day on main and about 1,450 PRs opened in the last 7 days. ORBIS cannot review everything. Start from upstream's own triage signals (`review queue`, `unblocks title`, `Fixes #` references) and from the duplicate clusters (ORB-004, ORB-009, ORB-010).
3. **Handle the duplicates before anything else.** In each cluster that shares a `Fixes #` reference, pick one PR. Treat the 212-PR errno series as a single decision (ORB-005).
4. **Keep upstream's GPU evidence rule.** Integrate agc or shader PRs only with a validation-layer diff and a GPU record (ORB-008). Keep at least one real AMD Polaris/GCN machine, because open Polaris and pre-RDNA2 issues exist (#9759, #15316) and CI only uses lavapipe (ORB-007).
5. **Pin everything.** Pin the upstream SHA, the submodule SHAs and the Mesa version for every baseline, so failures can be attributed (ORB-002, ORB-014).
6. **Resolve the FFmpeg download** before claiming a source-only build (ORB-001). **Send the licence inventory to human review** (ORB-012).
7. **Disclosure.** Every ORBIS change should carry `AI-assisted: yes`, follow upstream's wording, and state clearly that ORBIS is not upstream and is not endorsed by upstream or Sony (ORB-013).
