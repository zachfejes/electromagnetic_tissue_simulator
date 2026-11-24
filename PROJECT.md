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

## Physics & modelling

### Governing equations

- Electrostatics (FR-3) and magnetostatics (FR-4).
- Low-frequency (quasi-static) time-harmonic fields for FR-5 (D-2): eddy currents, inductive coupling and skin effect. Displacement currents and wave propagation are not modelled in v1.

### Tissue models & material properties

- v1 materials are general: each has electrical and magnetic properties that may depend on frequency and temperature (FR-8). Materials are linear and may be isotropic or anisotropic; dependence is given by a formula or an interpolated table (D-17). Temperature is an input (D-16).
- Biological tissue (layered tissue, published tissue property data, dispersion models) is the eventual target and builds on this material model.

### Numerical methods

- **Finite element method** on tetrahedral meshes (D-14), first-order elements in v1 (D-15).
- **Why a volume mesh is always needed.** Scripted geometry and STEP files describe surfaces exactly, but the unknown fields exist throughout the volumes (inside conductors and in the surrounding media). FEM breaks each volume into tetrahedra and integrates over each one with numerical quadrature. The exact surfaces define where material regions meet, and the mesher places element faces on them. First-order tetrahedra approximate curved surfaces with flat facets; this geometric error shrinks as the mesh is refined, and higher-order (curved) elements remove most of it later.

### Validation & verification

- Every physics feature has tests against analytical solutions (NFR-7). Candidate benchmarks: charged sphere, coaxial cable, infinite straight wire, circular current loop, conducting sphere in an alternating field, skin depth in a conducting half-space.
- Each benchmark states its error tolerance and checks that the error decreases as the mesh is refined.

## Architecture

- **Core libraries** (namespace `ets`), with no dependency on the simulator (NFR-3):
  - `ets::core`: math, mesh data structures and `.msh` reading;
  - `ets::physics`: materials and field formulations;
  - `ets::solvers`: assembly and linear solves.
- **Simulator** `ets-sim`: a command-line tool built on the core (FR-9).
- **Meshing**: Gmsh, run as a separate program (D-21).

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

Tools and compiler features (not linked libraries): Gmsh (separate program, D-21), OpenMP (compiler feature, D-27), Doxygen (D-23), clang-tidy and clang-format (D-9), llvm-cov (D-7).

## Environments

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

- The coverage threshold is enforced inside the workflow, not by the ruleset. The ruleset only sees whether the `coverage` check passed.
- The test framework is still to be chosen and approved (D-6).
- Ruleset setup (**Settings → Rules → Rulesets**, targeting the default branch):
  - Require a pull request before merging, with 0 required approvals. PRs opened from Claude sessions are authored as the repo owner, who can't approve their own PR; the owner merging is the approval.
  - Restrict deletions and block force pushes.
  - Require status checks to pass: `build-gcc`, `build-clang`, `unit-tests`, `coverage`, `lint`. Each job must have run at least once before GitHub offers it as a required check.
  - Optional: require branches to be up to date before merging.
  - Optional: require code scanning results (CodeQL). Needs a CodeQL workflow, and GitHub Advanced Security on a private repo.

### Dependency updates

## Code style & conventions

### Code

### Tests

### Copy and docs

## Milestones

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
