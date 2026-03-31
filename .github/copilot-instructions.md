# Copilot Repository Instructions

## Introduction
This file provides custom instructions to help Copilot understand how the LCECMD project is structured, how to build it, and how to interact with the codebase efficiently.  
The goal is to help Copilot minimize build issues, reduce exploratory commands, and improve accuracy when generating or modifying code.

---

## 1. Repository Summary
- **Purpose:** LCECMD is the source repository for an Ultra Rope Monitoring system. It provides the command-line and control logic for monitoring rope health, tension, and related metrics in elevator/lift systems.
- **Tech Stack:** To be documented as the codebase evolves (languages, frameworks, and build tools will be listed here).
- **Project Type:** CLI tool / embedded monitoring system.
- **Repository Size:** Small (actively growing).

---

## 2. Build & Execution Instructions

### **2.1 Bootstrap**
Before building, ensure the following are in place:
- Install any required dependencies (documented per language/framework as the project grows).
- Set required environment variables (to be specified as the project matures).
- Initialize any submodules: `git submodule update --init --recursive`

### **2.2 Build**
- Build commands will be documented here once a build system (e.g., `Makefile`, `CMakeLists.txt`, `package.json`) is introduced.
- Follow any order requirements noted in the build system files.

### **2.3 Test**
- Test commands will be documented once a test framework is established.
- Expected output and test artifacts will be described here.

### **2.4 Run**
- Execution instructions will be added once the entry point and runtime configuration are defined.
- Input/output behavior will be described here.

### **2.5 Lint & Format**
- Linting and formatting tools will be documented as they are adopted.

### **2.6 Validation Pipelines**
- CI/CD pipeline documentation will be added as workflows are introduced.
- Pre-merge validation steps and artifacts will be listed here.

---

## 3. Project Layout

### **3.1 Key Directories**
*(Directories will be documented as they are created.)*
- `/src` — main source code (to be created)
- `/include` — header files (to be created)
- `/config` — configuration files (to be created)
- `/scripts` — helper scripts (to be created)
- `/tests` — unit & integration tests (to be created)
- `/docs` — documentation (to be created)

### **3.2 Important Files**
- `README.md` — project overview
- `.github/copilot-instructions.md` — this file; Copilot guidance

### **3.3 Architectural Notes**
- LCECMD monitors rope health parameters (e.g., tension, wear, deformation) for elevator/ultra rope systems.
- Major components and their inter-module relationships will be documented here as the codebase grows.
- Deprecated modules or locations to avoid will be noted here.

### **3.4 Dependencies**
- External systems, API endpoints, and hidden dependencies will be documented as they are introduced.

---

## 4. Validation & CI/CD Behavior
- CI/CD pipeline definitions and triggers will be documented here as workflows are added.
- Steps Copilot must always follow before merging (e.g., running tests, linting) will be specified here.
- Security and compliance requirements will be noted here.

---

## 5. Guidance for Copilot
- Prefer existing utilities instead of writing new ones.
- Follow project naming conventions (e.g., `LCECMD` prefix for top-level command identifiers).
- Never modify files in deprecated directories.
- Reference constants instead of writing magic numbers.
- When unsure, search only after using the information in this file.
- Trust these instructions first; search the codebase only if information is missing or incorrect.
- Do not introduce new external dependencies without documenting them in this file.

---

## 6. Repository Inventory
- `README.md` — top-level project description ("LCECMD — Ultra Rope Monitoring Source").
- `.github/copilot-instructions.md` — this Copilot guidance file.
- *(Additional files and directories will be inventoried here as the project grows.)*

---

## 7. Final Notes for Copilot
- Follow these instructions for all code generation and modification tasks.
- Use build & test commands *exactly as documented* in this file.
- Explore the repository only if information is missing from or conflicts with this file.
- Keep this file up to date as the project evolves — update sections when new directories, dependencies, or workflows are added.
