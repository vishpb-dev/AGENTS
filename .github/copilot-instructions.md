# Copilot Repository Instructions — LCECMD

## Introduction

This file provides custom instructions to help Copilot understand how the LCECMD project is structured, how to build it, and how to interact with the codebase efficiently.  
The goal is to minimise build issues, reduce unnecessary exploratory commands, and improve accuracy when generating or modifying code.

---

## 1. Repository Summary

- **Purpose:** LCECMD is the source repository for an **Ultra Rope Monitoring** system. It provides the command-line and embedded-control logic for monitoring the health, tension, wear, and deformation of ropes used in elevator / lift installations.
- **Tech Stack:** To be finalised as the codebase matures. Expected stack: C / C++ (embedded firmware), Python or Bash (tooling/scripts), CMake or Make (build system).
- **Project Type:** CLI tool + embedded monitoring system.
- **Repository Size:** Small — actively growing from an initial scaffold.

---

## 2. Build & Execution Instructions

> **Note:** The project is in its bootstrapping phase. Commands below are pre-structured; fill them in and remove the placeholder text as each piece is introduced.

### 2.1 Bootstrap

Before building, complete the following steps **in order**:

1. Install required tools (document specific versions here once confirmed):
   - C/C++ toolchain (e.g., `gcc`/`g++` ≥ 12, or target cross-compiler)
   - CMake ≥ 3.20 (if using CMake)
   - Python ≥ 3.10 (for scripts/tooling)
2. Set required environment variables (add variables here as they are defined):
   ```bash
   # Example — replace with actual variables:
   # export LCECMD_TARGET=<target-platform>
   ```
3. Initialise submodules (if any):
   ```bash
   git submodule update --init --recursive
   ```
4. Install Python/script dependencies (update path and tool name as needed):
   ```bash
   # pip install -r requirements.txt
   ```

### 2.2 Build

> *(Update this section once a build system file is added to the repository.)*

```bash
# Example with CMake:
# cmake -B build -DCMAKE_BUILD_TYPE=Release
# cmake --build build

# Example with Make:
# make all
```

- Always build from the repository root.
- Note common errors and their mitigations here as they are discovered.

### 2.3 Test

> *(Update this section once a test framework is chosen and wired in.)*

```bash
# Example:
# cmake --build build --target test
# ctest --test-dir build --output-on-failure
```

- Describe expected output and generated test artifacts here.

### 2.4 Run

> *(Update this section once the main entry point is defined.)*

```bash
# Example:
# ./build/lcecmd --config config/default.yaml
```

- Describe runtime configuration options and expected input/output behaviour here.

### 2.5 Lint & Format

> *(Update this section once linting/formatting tools are adopted.)*

- Expected tools: `clang-format` for C/C++, `flake8`/`black` for Python.
- Commands (to be confirmed):
  ```bash
  # clang-format --dry-run -Werror src/**/*.c include/**/*.h
  # flake8 scripts/
  ```

### 2.6 Validation Pipelines

> *(Update this section once CI/CD workflows are introduced.)*

- Document GitHub Actions or GitLab CI jobs, triggers, and generated artefacts here.
- Pre-merge validation steps will be listed here once defined.

---

## 3. Project Layout

### 3.1 Key Directories

*(Directories are pre-declared below. Update statuses as they are created.)*

| Path | Status | Purpose |
|------|--------|---------|
| `/src` | Planned | Main source code (C/C++ implementation files) |
| `/include` | Planned | Public header files |
| `/config` | Planned | Configuration files (YAML, JSON, or INI) |
| `/scripts` | Planned | Helper and automation scripts |
| `/tests` | Planned | Unit and integration tests |
| `/docs` | Planned | Project documentation |
| `/.github` | Exists | GitHub configuration (Copilot instructions, CI workflows, agent files) |

### 3.2 Important Files

| File | Description |
|------|-------------|
| `README.md` | Top-level project description |
| `.github/copilot-instructions.md` | **This file** — Copilot guidance |
| `Makefile` / `CMakeLists.txt` | Build system entry point *(to be added)* |
| `.github/workflows/*.yml` | CI/CD pipeline definitions *(to be added)* |

### 3.3 Architectural Notes

- **Domain:** Elevator / lift rope health monitoring — sensor data ingestion, signal processing, threshold alerting.
- **Naming convention:**
  - Preprocessor macros and constants: `LCECMD_UPPER_SNAKE_CASE` (e.g., `LCECMD_MAX_TENSION_N`)
  - Public C functions and variables: `lcecmd_module_action` snake\_case with the `lcecmd_` lowercase prefix (e.g., `lcecmd_rope_read_tension`)
  - C++ classes/structs: `LcecmdModuleName` PascalCase (e.g., `LcecmdRopeMonitor`)
- **Module boundaries:** Keep hardware-abstraction (HAL) code strictly separated from business-logic (monitoring algorithms) and CLI/UI layers.
- **Deprecated locations:** None currently. Document deprecated modules here as the project evolves, and never modify code in those locations.

### 3.4 Dependencies

> *(Document external systems, third-party libraries, API endpoints, and hidden dependencies here as they are introduced.)*

- No external dependencies are declared yet.

---

## 4. Validation & CI/CD Behavior

> *(Document pipeline triggers, mandatory checks, and security requirements here as workflows are added.)*

- When CI workflows exist, Copilot must **always** run bootstrap steps before building or testing.
- Never merge code that breaks the build or causes test regressions.
- Security: do not commit secrets, credentials, or device-specific calibration data to this repository.

---

## 5. Guidance for Copilot

Follow these rules for all code-generation and modification tasks in this repository:

1. **Prefer existing utilities** — reuse helpers already present in `/src` or `/scripts` before writing new ones.
2. **Follow naming conventions:**
   - Macros / constants: `LCECMD_UPPER_SNAKE_CASE`
   - Public C functions and variables: `lcecmd_snake_case` (lowercase `lcecmd_` prefix)
   - C++ classes / structs: `LcecmdPascalCase`
3. **Use constants, not magic numbers** — define named constants in `/include` and reference them throughout the code.
4. **Respect module boundaries** — keep HAL, algorithm, and CLI/UI layers separate.
5. **Never modify deprecated directories** — these will be listed in §3.3 as the project evolves.
6. **Start from this file** — consult this file before searching the codebase. If the actual code conflicts with what is documented here, treat the code as authoritative and update this file to match.
7. **Do not introduce undocumented dependencies** — update §3.4 whenever a new external library or service is added.
8. **Keep this file current** — update all relevant sections and the §7 "Last updated" date whenever new directories, commands, or workflows are introduced.

---

## 6. Repository Inventory

| Path | Responsibility |
|------|---------------|
| `README.md` | Project overview ("LCECMD — Ultra Rope Monitoring Source") |
| `.github/copilot-instructions.md` | Copilot guidance (this file) |
| `.github/agents/my-agent_LCECMD.agent.md` | Custom Copilot agent definition for LCECMD |

*(Update this table as new files and directories are added.)*

---

## 7. Final Notes for Copilot

> **Last updated:** 2026-03-31 — keep this date current whenever sections are revised.

- Follow these instructions for **all** code generation and modification tasks.
- Use build, test, and lint commands **exactly as documented** in §2 — do not invent or guess commands.
- Consult this file first; if actual code contradicts what is written here, the code is authoritative — update this file to reflect reality.
- Keep this file up to date as the project grows.
