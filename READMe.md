# Developer Experience (DX) Report: Voxgig SDK Generator (`@voxgig/sdkgen`)

**Project:** Groq Cloud API Client SDK  
**License:** MIT  
**Target Surface:** TypeScript / Node.js  
**Evaluator:** Azeez Abidoye  
**Time-box:** ~30 Minutes

---

## 1. Executive Summary & Objective

The objective of this mini-task was to evaluate the developer experience (DX) of the **Voxgig SDK Generator (`@voxgig/sdkgen`)** by generating an open-source client SDK for an API not currently present in the Voxgig catalog.

I selected the **Groq Cloud API** (high-performance LPU AI inference). Because Groq exposes an OpenAI-compatible API surface without a one-click OpenAPI download button on its developer portal, I formulated a dedicated, standardized OpenAPI 3.0 specification (`groq-openapi.json`) covering core chat completions and models endpoints.

During the generation process, the generator demonstrated powerful architectural concepts (Aontu semantic modeling and Jostraca templating). However, the CLI onboarding path presented several notable friction points and edge-case bugs around parameter parsing, naming conventions, and silent defaults.

---

## 2. Chronological Log of Commands & Observed Behavior

| Step  | Command Executed                                      | Expected Behavior                                                      | Actual Behavior & Observed Friction                                                                                                                                               |
| :---- | :---------------------------------------------------- | :--------------------------------------------------------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **1** | `npm create @voxgig/sdkgen .`                         | Initialize interactive setup wizard in current directory.              | **Bug:** The CLI treated `.` as a string name prefix and concatenated `-sdk`, creating a hidden folder named `.-sdk`. No prompts appeared.                                        |
| **2** | `npm create @voxgig/sdkgen groq-sdk`                  | Scaffold a project named `groq-sdk` and prompt for OpenAPI spec.       | **Bug:** The tool blindly appended `-sdk` to the name, creating `groq-sdk-sdk`. It created `.sdk`, `.jostraca`, and `.github`, but generated no language targets or source files. |
| **3** | `ls -la groq-sdk-sdk`                                 | Expected standard package files (`package.json`, `src/`, `README.md`). | Only internal generator directories (`.sdk/`, `.jostraca/`, `.github/`) were present; no client code or build scripts were emitted.                                               |
| **4** | _Architecture Inspection_                             | Discovered internal workflow structure in `.sdk/model/`.               | Recognized that scaffolding only installs generator control files; actual compilation requires target definitions and specification flags.                                        |
| **5** | `npm create @voxgig/sdkgen -- groq -d ./openapi.json` | Generate target code directly from local spec.                         | Successfully ingested definition and resolved naming cleanly as `groq-sdk`.                                                                                                       |
| **6** | `cd groq-sdk/.sdk && npx voxgig-sdkgen target add ts` | Explicitly register TypeScript target and compile surfaces.            | Targeted TypeScript client generated cleanly with entity structures.                                                                                                              |

---

## 3. Deep-Dive: Developer Experience (DX) Friction Points & Bugs

### 🐛 Bug 1: Unsanitized Input Suffixing Resulting in Hidden Folders (`.-sdk`)

- **Finding:** Running `npm create @voxgig/sdkgen .` takes the positional argument `.` and performs a naive string concatenation: `${input}-sdk`.
- **Impact:** On Unix/macOS environments, this outputs a directory named `.-sdk`. Because it begins with a period, the folder is treated as hidden by standard shells and file explorers, confusing developers into thinking the command failed or produced nothing.

### 🐛 Bug 2: Redundant Suffix Appending (`groq-sdk-sdk`)

- **Finding:** When a developer naturally names their project with `-sdk` (e.g., `npm create @voxgig/sdkgen groq-sdk`), the tool unconditionally re-appends `-sdk`, resulting in `groq-sdk-sdk`.
- **Fix Suggestion:** Implement a simple regex check:
  ```javascript
  const cleanName = inputName.replace(/-sdk$/i, "");
  const targetDir = `${cleanName}-sdk`;
  ```
