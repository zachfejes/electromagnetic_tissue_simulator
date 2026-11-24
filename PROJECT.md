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
  - **FR-5.2** across a frequency sweep.
- **FR-6. Material boundaries.** Fields are modelled correctly across boundaries between different materials.
- **FR-7. Conductor geometry.** Users can specify the 3D geometry of a simulation:
  - **FR-7.1** by building it in code from primitives (e.g. wires, loops, boxes, cylinders) and combinations of them;
  - **FR-7.2** by importing an existing mesh.
- **FR-8. Variable material properties.** A material's electrical and magnetic properties can vary with frequency and with temperature.
- **FR-9. Command-line simulator.** A command-line tool reads a scenario file (geometry, materials, sources, frequencies) and writes field results in a standard format viewable in ParaView (e.g. VTK).

**Non-functional requirements**

- **NFR-1. Language.** The library is implemented in C++, for computational speed and efficiency.
- **NFR-2. Consumable as a dependency.** Other projects can add the library as a dependency and call it.
- **NFR-3. Layering.** The core math and physics code is a separate library from the simulator, with no dependency from the core on the simulator.
- **NFR-4. Test-driven development.** All code is developed test-first, following the project's TDD and testing skills.
- **NFR-5. Documentation.** Public classes and functions are documented.
- **NFR-6. Merge gates.** A pull request can merge to `main` only when CI shows:
  - **NFR-6.1** 100% line coverage;
  - **NFR-6.2** 100% branch coverage;
  - **NFR-6.3** no lint issues;
  - **NFR-6.4** a build that compiles without issues.
- **NFR-7. Verified accuracy.** Every physics feature is tested against known analytical solutions, each with a stated error tolerance and a check that the error shrinks as the mesh is refined.
- **NFR-8. License.** The library is released under the Apache-2.0 license.

### Out of scope

These are out of scope for v1. Most are listed under [Future ideas](#future-ideas).

- Biological tissue material libraries and tissue-specific models (the eventual goal, but after the general library).
- Full-wave electromagnetics (radiation, wave propagation) and time-domain transient solves.
- 2D and rotationally symmetric solvers.
- A graphical interface or interactive viewer.
- Python bindings.
- Writing our own mesher or general-purpose linear algebra.

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

## Physics & modelling

### Governing equations

- Electrostatics (FR-3) and magnetostatics (FR-4).
- Low-frequency (quasi-static) time-harmonic fields for FR-5 (D-2): eddy currents, inductive coupling and skin effect. Displacement currents and wave propagation are not modelled in v1.

### Tissue models & material properties

- v1 materials are general: each has electrical and magnetic properties that may depend on frequency and temperature (FR-8).
- Biological tissue (layered tissue, published tissue property data, dispersion models) is the eventual target and builds on this material model.

### Numerical methods

### Validation & verification

- Every physics feature has tests against analytical solutions (NFR-7). Candidate benchmarks: charged sphere, coaxial cable, infinite straight wire, circular current loop, conducting sphere in an alternating field, skin depth in a conducting half-space.
- Each benchmark states its error tolerance and checks that the error decreases as the mesh is refined.

## Architecture

- **Core libraries** (math, mesh, materials, physics, solvers): no dependency on the simulator (NFR-3).
- **Simulator**: a command-line tool built on the core (FR-9).

## Technical plan

- C++20, built with CMake, published as a CMake package (D-10).

### Approved dependencies

None yet. Each library must be recommended, described and approved before use (D-6).

## Environments

## Engineering workflow

### Branching

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
- Full-wave electromagnetics and time-domain transient solves.
- 2D and rotationally symmetric solvers for speed.
- Python bindings (e.g. via nanobind).
- Package-manager listings (vcpkg, Conan).
- macOS and Windows CI.
- An interactive viewer or GUI.
