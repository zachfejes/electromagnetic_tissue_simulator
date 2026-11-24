# Electromagnetic Tissue Simulator — Project Plan

## Goals

- **G1. A reusable research library.** The primary product is a library of reusable, tested, well-documented and accurate methods, classes and functions for electromagnetic simulation, used from code. Results must be accurate and reproducible enough to compare against measurements or published work.
- **G2. Usable by others.** Other projects can include the library and use it for their own electromagnetic calculations.
- **G3. Physics core separate from the simulator.** The core math and physics libraries are distinct from the simulator built on them, and do not depend on it.
- **G4. Interactive tools come later.** A viewer or interactive app may be built on top of the library later, but it is not part of the initial scope and must not shape the core design.

## Scope

### In scope

Requirements are numbered so tests, issues and PRs can refer to them. Each will be refined into testable acceptance criteria as the design settles.

**Functional requirements**

- **FR-1. Conductor materials.** A simulation can contain conductors made of different materials, each with its own electrical properties.
- **FR-2. Surrounding media.** A simulation can specify the materials around the conductor(s), each with its own electrical and magnetic properties.
- **FR-3. Steady-state electric fields.** The library computes the electric field for steady-state (static) sources.
- **FR-4. Steady-state magnetic fields.** The library computes the magnetic field for steady-state (DC) currents.
- **FR-5. Dynamic fields.** The library computes electric and magnetic fields driven by time-varying circuits, using a low-frequency (quasi-static), time-harmonic formulation (see D-2), including:
  - **FR-5.1** at points throughout one period of a set frequency;
  - **FR-5.2** across a frequency sweep;
  - **FR-5.3** driven by a prescribed voltage or current at conductor terminals, or by coils defined as a wire path carrying a current or as a wound region with a current density.
- **FR-6. Material boundaries.** Fields are modelled correctly across boundaries between different materials.
- **FR-7. Conductor geometry.** Users can specify the 3D geometry of a simulation:
  - **FR-7.1** by building it in code from primitives (e.g. wires, loops, boxes, cylinders) and combinations of them;
  - **FR-7.2** by importing an existing mesh.
- **FR-8. Variable material properties.** A material's electrical and magnetic properties can vary with frequency and with temperature:
  - **FR-8.1** materials can be isotropic or anisotropic (a property tensor), and are linear;
  - **FR-8.2** dependence on frequency and temperature can be given as a formula or as tabulated data with interpolation;
  - **FR-8.3** temperature is an input, either a single value or a value per region; it is not computed.
- **FR-9. Command-line simulator.** A command-line tool reads a scenario file (geometry, materials, sources, frequencies) and writes field results in a standard format viewable in ParaView (e.g. VTK).
- **FR-10. Derived quantities.** Besides E, B, H and current density J, the library computes resistance, inductance, capacitance, impedance across a frequency sweep, and power loss density.

**Non-functional requirements**

- **NFR-1. Language.** The library is implemented in C++, for computational speed and efficiency.
- **NFR-2. Consumable as a dependency.** Other projects can add the library as a dependency and call it.
- **NFR-3. Layering.** The core math and physics code is a separate library from the simulator, with no dependency from the core on the simulator.
- **NFR-4. Test-driven development.** All code is developed test-first, following the project's TDD and testing skills.
- **NFR-5. Documentation.** Every public class and function is documented, and the documentation explains the physics behind the code, not only what it does:
  - **NFR-5.1** API docs are generated with Doxygen, and CI fails if any public item is undocumented;
  - **NFR-5.2** physics code states the governing equations it implements (written in LaTeX, rendered by Doxygen) and how the code relates to them.
- **NFR-6. Merge gates.** A pull request can merge to `main` only when CI shows:
  - **NFR-6.1** 100% line coverage;
  - **NFR-6.2** 100% branch coverage;
  - **NFR-6.3** no lint issues;
  - **NFR-6.4** a build that compiles without issues.
- **NFR-7. Verified accuracy.** Every physics feature is tested against known analytical solutions, each with a stated error tolerance and a check that the error shrinks as the mesh is refined.
- **NFR-8. License.** The library is released under the Apache-2.0 license.
- **NFR-9. Units.** The API uses SI units throughout, stored as `double`, with the unit documented on every parameter and return value.
- **NFR-10. Problem size.** v1 handles problems of up to about one million unknowns on a single workstation, solved in minutes using multiple CPU threads. A benchmark suite tracks performance over time.
- **NFR-11. Capabilities in release notes.** Every release states the element order(s) it supports, along with its other solver capabilities.

### Out of scope

These are out of scope for v1. Most are listed under [Future ideas](#future-ideas).

- Biological tissue material libraries and tissue-specific models (the eventual goal, but after the general library).
- Full-wave electromagnetics (radiation, wave propagation) and time-domain transient solves.
- 2D and rotationally symmetric solvers.
- A graphical interface or interactive viewer.
- Python bindings.
- Writing our own mesher or general-purpose linear algebra.
- Nonlinear materials (e.g. saturating iron cores).
- Computing temperature (coupled electro-thermal or bioheat simulation).
- Lumped circuit elements (resistors, capacitors, inductors) coupled to the field model.
- Higher-order (curved) finite elements.

## Decisions

| # | Decision | Why |
|---|---|---|
| D-1 | v1 is a general-purpose EM library; biological tissue comes later. | A general, reusable core is the foundation tissue modelling needs. |
| D-2 | Dynamic fields use a low-frequency (quasi-static), time-harmonic formulation. | Covers eddy currents, inductive coupling and skin effect, and gives single-frequency and sweep results directly. Full-wave and transient solves come later. |
| D-3 | Problems are solved in full 3D from the start. | Realistic conductor geometry needs 3D. |
| D-4 | Geometry can be built in code or imported as a mesh; meshing uses an existing tool. | Writing a mesher is a large, separate project. |
| D-5 | The simulator is a command-line tool reading scenario files and writing VTK output. | Fits a scripted research workflow; a GUI waits until later (G4). |
| D-6 | Third-party libraries may be used for generic parts, but each must be recommended, described and approved by the project owner before use. Approved libraries are listed under [Approved dependencies](#approved-dependencies). | Avoids reinventing proven code while keeping dependencies deliberate. |
| D-7 | Coverage is measured with Clang source-based coverage (llvm-cov). Exclusions must be marked in the code and justified in the PR. | gcov counts compiler-generated exception branches, which makes 100% branch coverage unreachable. |
| D-8 | "Compiles without issues" means zero warnings, with a strict warning set treated as errors. CI builds on Linux with GCC and Clang; macOS and Windows come later. | Two compilers catch portability problems early; Linux keeps CI cheap. |
| D-9 | Linting is clang-tidy plus clang-format, each with a config committed to the repo. | Standard C++ tools that run well in CI. |
| D-10 | C++20 and CMake; the library is published as a CMake package. Package-manager listings (vcpkg, Conan) come later. | Modern, well-supported, and what other C++ projects expect. |
| D-11 | Accuracy is proven against analytical solutions with stated tolerances and mesh-convergence checks (NFR-7). | Makes "accurate" testable. |
| D-12 | Apache-2.0 license. | Permissive, with an explicit patent grant. |
| D-13 | No Python bindings in v1, but the C++ API is designed so they are easy to add. | Keeps v1 focused without closing the door. |
| D-14 | The finite element method (FEM) on tetrahedral meshes: nodal elements for electrostatics and steady current flow, edge (Nédélec) elements for magnetostatics and eddy currents. | Handles mixed materials, material boundaries (FR-6) and arbitrary 3D geometry. Integral methods are weak with many materials; FDTD is full-wave and time-domain. |
| D-15 | v1 uses first-order elements; interfaces leave room for higher orders. Each release notes the supported order (NFR-11). | Simplest to get correct and to cover fully with tests. |
| D-16 | Temperature is an input in v1, not computed. | Coupled heating is a separate solver. |
| D-17 | Materials are linear, isotropic or anisotropic; frequency and temperature dependence is a formula or interpolated table. | Anisotropy is needed for tissue and cheap to support early; nonlinear materials need an iterative outer loop. |
| D-18 | v1 sources are prescribed terminal voltages or currents and coils (wire path or wound region). | Covers most low-frequency cases; lumped-circuit coupling is a large extra feature. |
| D-19 | The library computes derived quantities (R, L, C, impedance, power loss density) as well as fields. | Usually what a researcher wants, and several have analytical benchmarks. |
| D-20 | SI units, `double`, documented per parameter; no units library. | Avoids a dependency and compile-time cost. |
| D-21 | Gmsh is used only as a separate tool. The library reads `.msh` v4.1 files itself and never links Gmsh. | Gmsh is GPL; linking it would conflict with Apache-2.0 (D-12). |
| D-22 | Target: up to about one million unknowns on one workstation, in minutes, multithreaded, with a benchmark suite (NFR-10). | Realistic coils and layered geometries without a cluster; sets the linear-solver requirements. |
| D-23 | API docs are generated with Doxygen; undocumented public items fail CI; physics code documents its governing equations (NFR-5). | Keeps "documented" from slipping, and explains why the code does what it does. |
| D-24 | C++ namespace `ets`; CMake targets `ets::core`, `ets::physics`, `ets::solvers`; command-line tool `ets-sim`. | Short and consistent. |
| D-25 | Scenario files are TOML. | Comments and forgiving hand-editing; scenarios are mostly written by hand. |
| D-26 | VTK output (`.vtu`) is written by our own code, not the VTK library. | A few hundred testable lines instead of a huge dependency. |
| D-27 | CPU multithreading uses OpenMP. The linear-solver and assembly back ends sit behind interfaces, and the public API does not expose Eigen or OpenMP types. | No extra dependency; the interfaces keep a GPU back end possible later. |
| D-28 | Dependencies: use an installed copy if found, otherwise download a pinned version with CMake FetchContent. | Consumers can supply their own copies; a fresh clone builds with no setup. |
| D-29 | Semantic Versioning, with a `CHANGELOG.md` whose every release has a Capabilities section (element order, physics features, validated benchmarks). | Signals breaking changes to library users and satisfies NFR-11. |
| D-30 | Doxygen docs, with equations rendered by MathJax, are published to GitHub Pages from `main`. | Equations are far easier to read rendered. |
| D-31 | One short-lived branch per change (`feat/`, `fix/`, `docs/`, `chore/`), merged by squash. | One readable commit per change on `main`. |
| D-32 | v1 starts with Eigen's built-in solvers (direct for small problems; preconditioned CG or BiCGSTAB for larger ones). Benchmarks measure progress toward NFR-10; a dedicated preconditioner (likely hypre's AMS) is proposed for approval only if the numbers call for it. | Start simple and measure before adding a dependency. |
| D-33 | Magnetostatics and eddy currents use the A–V formulation (A on edges, V on conductor nodes) with a tree–cotree gauge. | Uniform treatment of any geometry, including coils with holes; the gauge keeps the system well-posed. |
| D-34 | Open space is truncated with an air box and simple boundary conditions on its outer surface; a test shows the error falling as the box grows. | Simple, standard and testable; better open-boundary methods come later. |
| D-35 | Gmsh physical group names link meshes to scenarios: volume groups name materials, surface groups name terminals and boundaries. Unnamed or unused groups are rejected with a clear error. | Readable meshes and scenarios; mismatches are caught before an expensive solve. |
| D-36 | The public API throws exceptions for invalid input, with a message saying what is wrong and where. Inner numerical code assumes valid input and uses debug-build assertions. | Idiomatic C++; every `throw` is a branch the tests must cover. |
| D-37 | Repository layout as under [Architecture](#architecture). | Public headers separate from implementation; unit and validation tests separately runnable. |
| D-38 | All unit and validation tests run on every PR and count toward coverage; validation uses coarse meshes. A nightly job on `main` runs full mesh-refinement studies and performance benchmarks. | Every PR proves accuracy cheaply; expensive checks still run daily. |
| D-39 | CI runs in a pinned Docker image with GCC 13+, Clang 17+ and matching clang-tidy, clang-format and llvm-cov; the same image is available as a dev container. | A PR can't fail because a runner changed; local results match CI. |
| D-40 | Naming: `PascalCase` types, `camelCase` functions and variables, trailing underscore for private members (`value_`), `SCREAMING_CASE` for constants. clang-format uses LLVM style with a 100-column limit. Both are enforced by tooling. | Machine-checked, so style never comes up in review. A trailing underscore can never collide with names the C++ standard reserves. |
| D-41 | Enum values `PascalCase`; macros avoided, and where unavoidable `SCREAMING_CASE` with an `ETS_` prefix; namespaces lowercase; one main class per file, file named after it in `PascalCase` with `.hpp`/`.cpp`; files kept as brief as possible. | Short, single-class files make the file name an unambiguous pointer to the code. |
| D-42 | Code-built geometry (FR-7.1) works by generating a Gmsh `.geo` script and running the `gmsh` program to produce the `.msh`. A tiny built-in mesh generator covers simple test shapes so unit tests don't need Gmsh. | Gmsh quality and exact surfaces without linking it (D-21). |
| D-43 | The documentation gate (NFR-5.1) is its own CI job, `docs`, running Doxygen with warnings as errors. | A separate job makes a documentation failure obvious. |
| D-44 | Renovate keeps pinned versions (CMake, Docker image, GitHub Actions) up to date; each update is a PR that must pass every gate. | Dependabot can't read CMake pins. |
| D-45 | No outside contributions for now; `CONTRIBUTING.md` comes when the API stabilises. | A contribution process needs a stable API to contribute to. |
| D-46 | Releases are 0.x at milestone boundaries, starting with 0.1 after milestone 3. 1.0 comes when FR-1 to FR-10 are implemented and validated; the API is then frozen under Semantic Versioning. | 0.x signals an unstable API; 1.0 marks all requirements met. |
| D-47 | Only `.geo` scripts are committed; CI generates `.msh` files with the pinned Gmsh and caches them by script content and Gmsh version. Exception: a few tiny hand-checked `.msh` fixtures for the `.msh` reader's unit tests. | Meshing coarse models takes under a second, while committed meshes (0.4–14 MB each) would bloat git history; a script and its mesh can't drift apart. |
| D-48 | D-33 (tree–cotree gauge) is the default, but is re-checked with benchmarks at milestone 5 against leaving A ungauged with a consistent right-hand side and the AMS preconditioner. | The ungauged route often converges faster; decide with numbers. |

## Physics & modelling

### Governing equations

- Electrostatics (FR-3) and magnetostatics (FR-4).
- Low-frequency (quasi-static) time-harmonic fields for FR-5 (D-2): eddy currents, inductive coupling and skin effect. Displacement currents and wave propagation are not modelled in v1.
- Magnetostatics and eddy currents use the A–V formulation with a tree–cotree gauge (D-33).
- **Why a gauge is needed.** Only B = ∇×A is physical, and ∇×(∇φ) = 0 for any scalar φ, so A + ∇φ gives the same field as A. With edge elements this makes the system matrix singular: every discrete gradient is a solution of the homogeneous problem.
- **Tree–cotree gauge.** Treat the mesh's nodes and edges as a graph. A spanning tree connects every node with no loops; the remaining edges form the cotree, and each cotree edge closes exactly one loop with tree edges. Setting A = 0 on tree edges removes exactly the gradient freedom, because a nodal function is fully determined by its differences along tree edges. The cotree edges keep the physical content: the circulation of A around each loop, which equals the magnetic flux through it. The result is a unique A and a non-singular system, at the cost of building one spanning tree. The exact variant used with V in conductors is derived in the physics docs at milestone 5.
- The domain is truncated by an air box with simple boundary conditions on its outer surface (D-34).

### Tissue models & material properties

- v1 materials are general: each has electrical and magnetic properties that may depend on frequency and temperature (FR-8). Materials are linear and may be isotropic or anisotropic; dependence is given by a formula or an interpolated table (D-17). Temperature is an input (D-16).
- Biological tissue (layered tissue, published tissue property data, dispersion models) is the eventual target and builds on this material model.

### Numerical methods

- **Finite element method** on tetrahedral meshes (D-14), first-order elements in v1 (D-15).
- **Linear solvers:** Eigen's built-in direct and iterative solvers to start; a dedicated preconditioner only if benchmarks show the need (D-32).
- **Candidate preconditioner: hypre (not approved).** [hypre](https://github.com/hypre-space/hypre) is Lawrence Livermore National Laboratory's library of scalable preconditioners, dual-licensed MIT / Apache-2.0. Two parts are relevant:
  - **BoomerAMG**, algebraic multigrid for scalar problems (electrostatics, steady current flow). Plain CG needs more iterations as the mesh is refined; multigrid keeps the iteration count nearly constant, so solve time grows roughly in proportion to problem size.
  - **AMS** (Auxiliary-space Maxwell Solver), for curl–curl problems on first-order edge elements (magnetostatics, eddy currents). Generic multigrid fails here because of the huge gradient null space (the freedom the gauge removes). AMS splits the error into a gradient part and a remaining part, maps each onto auxiliary nodal problems that BoomerAMG solves well, and needs only the discrete gradient matrix (edges to nodes) and the node coordinates. It handles non-conducting regions and ungauged systems.
  - Costs: a compiled C library (can be built without MPI), and matrices must be converted to hypre's format. Proposed for approval only if milestone 5–8 benchmarks justify it (D-32, D-48).
- **Why a volume mesh is always needed.** Scripted geometry and STEP files describe surfaces exactly, but the unknown fields exist throughout the volumes (inside conductors and in the surrounding media). FEM breaks each volume into tetrahedra and integrates over each one with numerical quadrature. The exact surfaces define where material regions meet, and the mesher places element faces on them. First-order tetrahedra approximate curved surfaces with flat facets; this geometric error shrinks as the mesh is refined, and higher-order (curved) elements remove most of it later.

### Validation & verification

- Every physics feature has tests against analytical solutions (NFR-7). Candidate benchmarks: charged sphere, coaxial cable, infinite straight wire, circular current loop, conducting sphere in an alternating field, skin depth in a conducting half-space.
- Each benchmark states its error tolerance and checks that the error decreases as the mesh is refined.
- A test shows the error caused by the air box falling as the box grows (D-34).
- PRs run validation on coarse meshes; the nightly job runs full refinement studies (D-38).
- Validation meshes are generated in CI from committed `.geo` scripts and cached (D-47).

## Architecture

- **Core libraries** (namespace `ets`), with no dependency on the simulator (NFR-3):
  - `ets::core`: math, mesh data structures and `.msh` reading;
  - `ets::physics`: materials and field formulations;
  - `ets::solvers`: assembly and linear solves.
- **Simulator** `ets-sim`: a command-line tool built on the core (FR-9).
- **Meshing**: Gmsh, run as a separate program (D-21). Physical group names link a mesh to its scenario (D-35).
- **Errors**: the public API throws exceptions for invalid input (D-36).

Repository layout (D-37):

```
include/ets/{core,physics,solvers}/   public headers
src/{core,physics,solvers}/           implementation
apps/ets-sim/                          command-line simulator
tests/unit/                            fast unit tests
tests/validation/                      analytical benchmarks (NFR-7)
benchmarks/                            performance benchmarks (NFR-10)
examples/                              sample meshes, scenarios, Gmsh scripts
docs/                                  physics notes and Doxygen config
```

## Technical plan

- C++20, built with CMake, published as a CMake package (D-10).

### Approved dependencies

Each library must be recommended, described and approved before use (D-6). Only the libraries below are approved.

| Library | Purpose | Used by | License | Approved |
|---|---|---|---|---|
| [Eigen](https://eigen.tuxfamily.org) | Dense and sparse linear algebra; built-in iterative and direct sparse solvers | Core libraries | MPL-2.0 (unmodified use is fine under Apache-2.0) | 2026-11-24 |
| [GoogleTest](https://github.com/google/googletest) (incl. GoogleMock) | Unit, parameterised and validation tests | Tests only | BSD-3-Clause | 2026-11-24 |
| [toml++](https://github.com/marzer/tomlplusplus) | Parsing TOML scenario files | `ets-sim` only | MIT | 2026-11-24 |
| [CLI11](https://github.com/CLIUtils/CLI11) | Command-line options and subcommands | `ets-sim` only | BSD-3-Clause | 2026-11-24 |
| [Google Benchmark](https://github.com/google/benchmark) | Timing assembly and solve phases (NFR-10) | `benchmarks/` only | Apache-2.0 | 2026-11-24 |

Tools and compiler features (not linked libraries): Gmsh (separate program, D-21), OpenMP (compiler feature, D-27), Doxygen (D-23), clang-tidy and clang-format (D-9), llvm-cov (D-7).

Services: [Renovate](https://github.com/apps/renovate) for dependency update PRs (D-44), approved 2026-11-24.

## Environments

- **CI**: GitHub Actions running in a pinned Docker image with GCC 13+, Clang 17+, clang-tidy, clang-format and llvm-cov (D-39).
- **Local development**: the same image as a dev container, so builds, lint and coverage match CI.
- **Nightly**: a scheduled job on `main` runs full refinement studies and performance benchmarks (D-38).
- **Docs**: Doxygen output published to GitHub Pages from `main` (D-30).

## Engineering workflow

### Branching

- One short-lived branch per change, named `feat/…`, `fix/…`, `docs/…` or `chore/…` (D-31).
- Every change reaches `main` through a pull request, merged by squash.

### CI gates on pull requests

Every pull request to `main` must pass these checks before it can merge (NFR-6). The checks are jobs in a CI workflow (`.github/workflows/ci.yml`, not yet written), and a branch ruleset on `main` makes them required.

| Check (job name) | What it does | Fails when |
|---|---|---|
| `build-gcc`, `build-clang` | Builds everything on Linux with a strict warning set treated as errors (D-8) | Any error or warning |
| `unit-tests` | Runs the full test suite, including the analytical benchmarks | Any test fails |
| `coverage` | Runs the tests with Clang source-based coverage (D-7) | Line coverage or branch coverage is below 100% |
| `lint` | Runs clang-tidy and checks formatting with clang-format (D-9) | Any issue is reported |
| `docs` | Runs Doxygen with warnings as errors (D-43) | Any public item is undocumented, or any other Doxygen warning |

- The coverage threshold is enforced inside the workflow, not by the ruleset. The ruleset only sees whether the `coverage` check passed.
- Tests use GoogleTest. Unit and validation tests both run on every PR and count toward coverage (D-38).
- Ruleset setup (**Settings → Rules → Rulesets**, targeting the default branch):
  - Require a pull request before merging, with 0 required approvals. PRs opened from Claude sessions are authored as the repo owner, who can't approve their own PR; the owner merging is the approval.
  - Restrict deletions and block force pushes.
  - Require status checks to pass: `build-gcc`, `build-clang`, `unit-tests`, `coverage`, `lint`, `docs`. Each job must have run at least once before GitHub offers it as a required check.
  - Optional: require branches to be up to date before merging.
  - Optional: require code scanning results (CodeQL). Needs a CodeQL workflow, and GitHub Advanced Security on a private repo.

### Dependency updates

- Renovate opens a PR for each new version of a pinned dependency, the Docker image or a GitHub Action (D-44). Update PRs pass the same gates as any other change.
- Any new library still needs explicit approval (D-6).

## Code style & conventions

### Code

- C++20 (D-10).
- Naming (D-40), enforced by clang-tidy's `readability-identifier-naming`:
  - types: `PascalCase`;
  - functions and variables: `camelCase`;
  - private members: trailing underscore, e.g. `value_`;
  - enum values: `PascalCase`, e.g. `ElementOrder::First`;
  - namespaces: lowercase, e.g. `ets::physics`;
  - macros: avoided; where unavoidable, `SCREAMING_CASE` with an `ETS_` prefix;
  - constants: `SCREAMING_CASE`.
- Formatting: clang-format, LLVM style, 100-column limit (D-40).
- Files: one main class per file, named after it in `PascalCase` (`TetrahedralMesh.hpp`, `TetrahedralMesh.cpp`), and kept as brief as possible (D-41).
- SI units, documented on every parameter and return value (NFR-9).
- Invalid input to the public API throws an exception (D-36).

### Tests

### Copy and docs

## Milestones

Each milestone is a set of PRs, each one test-first.

0. **Foundations.** Copy and adapt the owner's `tdd` and `testing` skills; CMake skeleton; CI with all six gates; pinned Docker image and dev container; ruleset updated with the six required checks. No physics.
1. **Mesh in, results out.** Read `.msh` v4.1 with physical groups; validate meshes against scenarios (D-35); write `.vtu`; tiny mesh generator for test shapes.
2. **Materials.** Linear isotropic and anisotropic materials, with frequency and temperature dependence by formula or table.
3. **Electrostatics.** Nodal FEM; validated against a charged sphere and a coaxial cable; capacitance. **Release 0.1.**
4. **Steady current flow.** Conduction in conductors; resistance.
5. **Magnetostatics.** Edge elements with a tree–cotree gauge; validated against an infinite wire and a current loop; inductance.
6. **Eddy currents.** Time-harmonic A–V; fields over one period; frequency sweeps; impedance; power loss; validated against skin depth and a conducting sphere.
7. **Simulator.** `ets-sim` with TOML scenarios; code-built geometry via Gmsh.
8. **Performance.** Benchmarks toward one million unknowns; preconditioner decision (D-32).

Releases: 0.x at milestone boundaries; 1.0 when FR-1 to FR-10 are implemented and validated (D-46).

## Open questions

- **Copy the owner's `tdd` and `testing` skills into this repo.** They live on the owner's computer, not in personal_website. Connect that computer to a session (or paste each `SKILL.md`), then copy and adapt them for C++ in a standalone PR. NFR-4 depends on them.
- **How Gmsh is used.** Gmsh is GPL-2.0-or-later. Linking its API into the library would make the combined work GPL, which conflicts with Apache-2.0 (D-12). Using Gmsh only as a separate tool, with the library reading `.msh` files itself, avoids linking.

## Future ideas

- Biological tissue: layered tissue models, published tissue property data, dispersion models.
- Coupled electro-thermal and bioheat simulation (computed temperature).
- Nonlinear materials.
- Lumped circuit elements coupled to the field model.
- Higher-order (curved) elements.
- Full-wave electromagnetics and time-domain transient solves.
- 2D and rotationally symmetric solvers for speed.
- Python bindings (e.g. via nanobind).
- Package-manager listings (vcpkg, Conan).
- macOS and Windows CI.
- An interactive viewer or GUI.
- GPU acceleration (e.g. CUDA, HIP, SYCL or OpenMP offload; GPU sparse solvers) as an alternative back end behind the solver and assembly interfaces (D-27).
