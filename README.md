# AutoGrader

**Repository:** <https://github.com/Dynamic-Vector/autograder>

**A reproducible, rule-driven grading system for C programs---with a
local web interface and optional LLM-assisted asset generation.**

AutoGrader compiles and executes C submissions in a constrained
environment, evaluates them with independent analyzers, and produces a
weighted score with actionable feedback. Instructors define grading
criteria in `rules.json`. When enabled, automatic asset generation can
prepare missing test inputs, corner cases, input generators, and
candidate reference oracles from the problem metadata and available
examples.

The project provides:

-   A browser-based interface for grading, asset inspection, and
    history.
-   A Docker-based Linux toolchain for consistent grading across host
    operating systems.
-   Local LLM integration through Ollama (required).
-   JSON and plain-text grading reports.
-   Persistent generated assets and grading history.
-   A command-line interface for batch grading, validation, and
    reproducibility checks.

> **Important:** LLM-generated assets are candidates, not ground truth.
> AutoGrader validates generated assets before using them. Oracle
> agreement is evidence, not a mathematical proof of correctness. Review
> generated assets and reports before using grades for high-stakes
> decisions.

------------------------------------------------------------------------

## Contents

1.  [Overview](#1-overview)
2.  [How grading works](#2-how-grading-works)
3.  [Prerequisites](#3-prerequisites)
4.  [Install and run on Windows (PowerShell and CMD)](#4-install-and-run-on-windows-powershell-and-cmd)
5.  [Install and run on Linux](#5-install-and-run-on-linux)
6.  [Use the web interface](#6-use-the-web-interface)
7.  [Automatic asset generation](#7-automatic-asset-generation)
8.  [Metrics and analyzers](#8-metrics-and-analyzers)
9.  [Reports, generated files, and
    persistence](#9-reports-generated-files-and-persistence)
10. [Rules and problem inputs](#10-rules-and-problem-inputs)
11. [CLI reference](#11-cli-reference)
12. [Configuration](#12-configuration)
13. [Security and reproducibility](#13-security-and-reproducibility)
14. [Troubleshooting](#14-troubleshooting)
15. [Development and tests](#15-development-and-tests)

------------------------------------------------------------------------

## 1. Overview

AutoGrader evaluates a C program against an instructor-defined set of
metrics. A typical grading run:

1.  Loads and validates `rules.json`.
2.  Optionally prepares missing assets.
3.  Compiles the submission.
4.  Runs a smoke test and the applicable analyzers.
5.  Collects metric scores, evidence, counterexamples, and limitations.
6.  Calculates the weighted grade.
7.  Writes JSON and text reports and records the run in the local
    database.

The same grading engine is used by the CLI and web interface.

### Main capabilities

-   **Submission grading:** Grade an individual C source file.
-   **Batch grading:** Grade a directory of student submissions.
-   **Automatic asset preparation:** Generate missing assets when the
    user enables this option in the UI or passes `--generate-assets` to
    the CLI.
-   **Independent oracle validation:** Candidate reference programs are
    screened, compiled, and checked against instructor-provided examples
    where available.
-   **Static and dynamic analysis:** Inspect compiler warnings, source
    structure, memory behavior, and runtime behavior.
-   **Originality analysis:** Compare normalized code fingerprints
    against available submission and known-solution corpora.
-   **Reproducibility checks:** Compare report fingerprints across
    repeated runs.
-   **Local web UI:** Use the Grade, Assets, and History views without
    needing a frontend build process.

------------------------------------------------------------------------

## 2. How grading works

The grading pipeline has six stages:

1.  **Load rules.** AutoGrader validates metric names, weights,
    parameters, and referenced paths. Metric weights should sum to
    `1.00`.
2.  **Prepare assets (optional).** If automatic generation is enabled,
    AutoGrader identifies missing assets and attempts to generate and
    validate them.
3.  **Compile and smoke-test.** The submission is compiled and run on
    the configured sample input. A compile failure or non-terminating
    smoke test can prevent execution-based metrics from running.
4.  **Run analyzers.** Applicable analyzers evaluate the submission in
    the sandbox. Source-only checks can still run when execution-based
    checks cannot.
5.  **Calculate and explain.** AutoGrader combines metric scores using
    their weights and reports findings, failing inputs, expected and
    actual outputs, explanations, and suggested fixes.
6.  **Save results.** JSON and TXT reports are written to the reports
    directory; the web UI's History view reads stored grading records
    from SQLite.

A metric that cannot be measured is reported as unavailable or not
performed with a reason. It should not be presented as a successful
test.

------------------------------------------------------------------------

## 3. Prerequisites

### Required

-   Git
-   Docker Desktop on Windows, or Docker Engine on Linux
-   A terminal: PowerShell on Windows; Bash on Linux
-   The repository and a working internet connection for the initial
    clone and image build

### Install and configure Ollama

Ollama provides a local model for LLM-assisted review and automatic
asset generation.

Install Ollama from <https://ollama.com/download>, then download the
model:

``` bash
ollama pull qwen2.5:7b
```

The model download is several gigabytes. Ensure sufficient disk space
and RAM.

### Docker toolchain

The Docker image is intended to provide a consistent Linux grading
environment, including tools such as GCC, Valgrind, Cppcheck, Lizard,
and sanitizers. Use Docker for grading that needs the pinned toolchain
rather than relying on host-specific tool versions.

------------------------------------------------------------------------

## 4. Install and run on Windows (PowerShell and CMD)

Windows users can follow either the **PowerShell** or **Command Prompt (CMD)** instructions below. Both use Docker Desktop and do not require Git Bash.

### 4.1 Install Docker Desktop and Git

1.  Install [Git for Windows](https://git-scm.com/downloads).
2.  Install [Docker
    Desktop](https://www.docker.com/products/docker-desktop/).
3.  Start Docker Desktop and wait until its engine is running.
4.  Open PowerShell.

Check Docker:

``` powershell
docker version
docker info
```

### 4.2 Clone the repository

Choose a directory for your projects, then run:

``` powershell
cd "$HOME\Projects"
git clone https://github.com/Dynamic-Vector/autograder.git autograder
cd autograder
```

If you have already cloned the repository, change into that existing
directory instead.

Confirm that the current directory contains the Dockerfile:

``` powershell
Get-ChildItem Dockerfile
```

### 4.3 Build the image

Run this once after cloning, and again whenever you change the Docker
image or its dependencies:

``` powershell
docker build -t autograder:latest .
```

Check that the image exists:

``` powershell
docker images autograder
```

### 4.4 Install and start Ollama

Install Ollama for Windows and download the model:

``` powershell
ollama pull qwen2.5:7b
```

If Ollama is already running as a Windows application, you normally do
not need to start a second server.

If you need to start it manually, close the existing Ollama server
first, then run this in a separate PowerShell window:

``` powershell
$env:OLLAMA_HOST = "0.0.0.0:11434"
ollama serve
```

Keep the Ollama window open. In another PowerShell window, check the
API:

``` powershell
curl.exe http://localhost:11434/api/tags
```

### 4.5 Start the web application

From the repository root, run the following command in **PowerShell**:

``` powershell
docker run --rm -it `
  -p 8000:8000 `
  -v "${PWD}:/work" `
  -w /work `
  -e AUTOGRADER_CACHE=/work/.autograder-cache `
  -e OLLAMA_HOST=http://host.docker.internal:11434 `
  autograder:latest serve --host 0.0.0.0 --port 8000
```

The volume mount makes the repository available inside the container at
`/work`. Reports and generated assets written under the mounted
directory remain available on Windows after the container stops.

Keep this terminal open while using the UI.

Open:

**http://localhost:8000**

### 4.6 Stop the application

Return to the terminal running AutoGrader and press:

``` text
Ctrl + C
```

The `--rm` option removes the stopped container. It does not remove the
project files, reports, or mounted cache directory.

### 4.7 Restart later

After the image has been built, start Docker Desktop, ensure Ollama is
running, open PowerShell in the repository root, and run the
same `docker run` command from section 4.5.

### 4.8 Command Prompt (CMD) alternative

Use this section if you prefer the classic Windows Command Prompt instead
of PowerShell. Open **Command Prompt** (`cmd.exe`). Start Docker Desktop
before running Docker commands.

**Clone the repository:**

``` bat
mkdir "%USERPROFILE%\Projects"
cd /d "%USERPROFILE%\Projects"
git clone https://github.com/Dynamic-Vector/autograder.git autograder
cd autograder
```

If the repository is already cloned, use `cd /d` to enter its folder
instead. Confirm that `Dockerfile` is present:

``` bat
dir Dockerfile
```

**Build the Docker image:**

``` bat
docker build -t autograder:latest .
docker images autograder
```

**Download and start Ollama.** Install Ollama for Windows,
then pull the model:

``` bat
ollama pull qwen2.5:7b
```

If Ollama is not already running as a Windows application, open a
separate CMD window and run:

``` bat
set OLLAMA_HOST=0.0.0.0:11434
ollama serve
```

Keep that window open. In another CMD window, test the API:

``` bat
curl.exe http://localhost:11434/api/tags
```

**Start the AutoGrader web UI.** In CMD, first change to the repository
root (the directory containing the `Dockerfile`), then run:

``` bat
docker run --rm -it ^
  -p 8000:8000 ^
  -v "%cd%:/work" ^
  -w /work ^
  -e AUTOGRADER_CACHE=/work/.autograder-cache ^
  -e OLLAMA_HOST=http://host.docker.internal:11434 ^
  autograder:latest serve --host 0.0.0.0 --port 8000
```

Open **http://localhost:8000** in your browser. Keep the CMD window open
while using AutoGrader. Press `Ctrl + C` in that window to stop it.

**Restart later:** Start Docker Desktop, make sure Ollama is running, open CMD in the repository root, and run the
same `docker run` command above. You do not need to clone or build the
image again unless the source or Docker image has changed.

------------------------------------------------------------------------

## 5. Install and run on Linux

The following commands use Ubuntu/Debian-style package management and
Bash.

### 5.1 Install Docker

Install Git:

``` bash
sudo apt update
sudo apt install -y git
```

Install Docker Engine using the official Docker instructions for your
Linux distribution:

<https://docs.docker.com/engine/install/>

Start and verify Docker:

``` bash
sudo systemctl enable --now docker
sudo docker version
```

If you configure non-root Docker access, follow Docker's official
post-installation instructions. Otherwise, prefix Docker commands with
`sudo`.

### 5.2 Clone the repository

``` bash
mkdir -p ~/Projects
cd ~/Projects
git clone https://github.com/Dynamic-Vector/autograder.git autograder
cd autograder
```

### 5.3 Build the image

``` bash
docker build -t autograder:latest .
```

If your account does not have permission to access Docker:

``` bash
sudo docker build -t autograder:latest .
```

### 5.4 Install and start Ollama

Install Ollama using the official Linux instructions:

<https://ollama.com/download/linux>

Pull the model:

``` bash
ollama pull qwen2.5:7b
```

If Ollama is managed as a system service, check it:

``` bash
systemctl status ollama
```

For a Docker container to reach Ollama on the Linux host, Ollama must
listen on a host interface reachable from Docker. If you manage Ollama
manually, start it with:

``` bash
OLLAMA_HOST=0.0.0.0:11434 ollama serve
```

Keep it running. In another terminal, test the API:

``` bash
curl http://localhost:11434/api/tags
```

Restrict access to the Ollama port using your host firewall. Do not
expose it to untrusted networks.

### 5.5 Start the web application

From the repository root:

``` bash
docker run --rm -it \
  -p 127.0.0.1:8000:8000 \
  -v "$(pwd):/work" \
  -w /work \
  -e AUTOGRADER_CACHE=/work/.autograder-cache \
  -e OLLAMA_HOST=http://host.docker.internal:11434 \
  --add-host=host.docker.internal:host-gateway \
  autograder:latest serve --host 0.0.0.0 --port 8000
```

Open:

**http://localhost:8000**

Keep the terminal open while using the UI. Press `Ctrl + C` to stop the
application.

If Docker requires root privileges on your system, use
`sudo docker run ...` with the same arguments. Ensure the mounted
project directory remains writable by the user who needs to access
generated files and reports.

------------------------------------------------------------------------

## 6. Use the web interface

The web UI is served locally by the application. It is a static page
backed by the Python standard-library HTTP server; it does not require
Node.js, npm, or a separate frontend build.

### 6.1 Grade tab

Use the Grade tab to:

1.  Select a problem's `rules.json` from the repository or upload a
    rules file, depending on the workflow supported by the UI.
2.  Upload or select the student's `.c` submission.
3.  Enable **Generate missing assets** when you want AutoGrader to
    prepare missing test assets.
4.  Choose the LLM provider and model if the UI exposes those controls.
5.  Use the LLM connection check before starting generation when
    available.
6.  Start grading and review the result.

The result view presents the overall grade, metric-level scores and
statuses, flags, analyzer findings, failing inputs, expected versus
actual output, explanations, and suggestions.

Use the report controls to download:

-   **TXT report:** A readable report for instructors or students.
-   **JSON report:** The structured machine-readable report.

### 6.2 Assets tab

Use the Assets tab to inspect asset preparation. It distinguishes:

-   **Instructor-provided:** Files supplied with the problem.
-   **Generated:** Files created during automatic preparation, including
    rejected or skipped assets and their reasons.
-   **Validated and used for grading:** Assets that passed validation
    and were supplied to the relevant analyzers.

The asset details include generation decisions, oracle candidate
verdicts, validation outcomes, and file locations. Inspect rejected
assets and warnings rather than assuming every LLM response was
accepted.

### 6.3 History tab

The History tab displays stored grading runs. Use it to review previous
grades and compare reproducibility fingerprints where available.

------------------------------------------------------------------------

## 7. Automatic asset generation

Automatic asset generation is optional and is triggered by enabling
**Generate missing assets** in the UI or passing `--generate-assets` to
a CLI grading command.

### 7.1 Minimal input workflow

The intended minimal workflow is:

``` text
submission.c
rules.json
```

A separate `asset_generation` section in `rules.json` is not required
for the default automatic workflow.

AutoGrader uses the available problem metadata and examples---such as
the problem identifier/title, limits, sample input/output, configured
metrics, and any available statement or instructor assets---to infer the
problem format.

If the available information is ambiguous or insufficient, generation or
validation may fail. The report should explain why. A problem title
alone may not uniquely define input format, output format, constraints,
or special-case semantics; provide reliable sample input/output or an
instructor statement when possible.

### 7.2 What may be generated

Depending on the metrics and information available, the preparation
pipeline can create:

-   A structured input specification.
-   A deterministic input generator.
-   Boundary and corner-case inputs.
-   Candidate oracle/reference programs.
-   Expected outputs for inputs where a trustworthy oracle is available.
-   Known-solution samples for originality comparison.
-   A validation report and generation manifest.

Not every asset is required for every problem. The pipeline should
generate only assets needed by configured analyzers.

### 7.3 Oracle validation and trust

The student's submission must never be used as its own answer key.

Generated oracle candidates are independently screened, compiled, and
checked against trusted instructor examples where available. Expected
outputs are used only when the configured trust and agreement
requirements are met. If candidates disagree or validation fails, the
affected tests or metrics may be marked partial or not performed.

LLM consensus does not prove that an oracle is correct. For high-stakes
grading, instructors should review generated specifications, oracle
decisions, and representative test cases.

### 7.4 Generated asset location

Generated assets are stored under the project cache:

``` text
.autograder-cache/
└── generated/
    └── <problem>/
        └── <generation_id>/
            ├── manifest.json
            ├── validation/
            ├── spec/
            ├── generators/
            ├── tests/
            ├── edge_cases/
            ├── inputs/
            ├── oracles/
            ├── reference/
            └── known/
```

The exact files depend on which assets were requested and successfully
generated. The instructor's problem folder is not the destination for
generated assets. With the project mounted into Docker, this cache is
visible in the host project directory.

Generated bundles are cached and reused when their dependencies have not
changed. Changes to rules, instructor assets, model or generation
settings, prompts, generator version, or compiler can result in a new
bundle.

------------------------------------------------------------------------

## 8. Metrics and analyzers

The configured metrics determine which analyzers run. Each analyzer may
require different inputs; a metric can be unavailable if its required
evidence or assets are missing.

  ------------------------------------------------------------------------
  Analyzer                 Purpose                 Typical requirements
  ------------------------ ----------------------- -----------------------
  `io_tests`               Checks program output   Test inputs and
                           against test            expected outputs, or a
                           expectations            trusted reference

  `compiler_warnings`      Reports compiler        C compiler
                           warnings and            
                           diagnostics             

  `static_analysis`        Inspects source-level   Source; Cppcheck where
                           issues, complexity, and enabled
                           selected static checks  

  `function_split`         Evaluates decomposition Source
                           into functions          

  `edge_case_runner`       Executes corner cases   Corner cases and a
                           and seeded random tests trusted
                           against trusted         oracle/reference;
                           expected outputs        generator for random
                                                   tests

  `complexity_estimator`   Estimates growth        Valid input generator
                           against a required      and requested sizes
                           complexity class        

  `sanitizer_run`          Checks for memory and   Compiler sanitizers;
                           undefined-behavior      Valgrind where
                           issues                  available

  `llm_review`             Evaluates readability   Reachable configured
                           using an LLM rubric     LLM

  `similarity_flag`        Compares code           Submission corpus
                           fingerprints against    and/or known-solution
                           available corpora       corpus

  `source_patterns`        Checks required or      Configured patterns
                           forbidden source        
                           patterns                
  ------------------------------------------------------------------------

### Understanding statuses

-   **Prepared:** Required generated assets passed validation and are
    available to the metric.
-   **Partial:** Some checks ran, while others were skipped with
    reasons.
-   **Not performed:** The metric could not run because required inputs
    or trustworthy expected outputs were unavailable.
-   **Generation failed:** An asset-generation or validation step
    failed. This is an infrastructure/preparation issue, not
    automatically a student-code failure.
-   **Suspect similarity:** Similarity evidence is insufficient for a
    confident plagiarism flag. Small programs can naturally share common
    code patterns.

Originality results should be interpreted with care. Similarity is a
signal for human review, not a standalone determination of misconduct.

------------------------------------------------------------------------

## 9. Reports, generated files, and persistence

### Project data

  ----------------------------------------------------------------------------
  Path                                     Contents
  ---------------------------------------- -----------------------------------
  `reports/`                               Text and JSON grading reports;
                                           batch summaries where applicable

  `.autograder-cache/llm/`                 Cached LLM responses

  `.autograder-cache/autograder.sqlite3`   Grading history, generated bundles,
                                           assets, and generation events

  `.autograder-cache/generated/`           Materialized generated asset
                                           bundles
  ----------------------------------------------------------------------------

These paths are inside the repository working directory when using the
Docker commands in this guide.

### Persistence

The Docker commands mount the current repository directory into `/work`.
This is important: files written to `/work/reports` or
`/work/.autograder-cache` inside the container are written to the
corresponding host directory.

The container itself is temporary because the commands use `--rm`. The
project files, reports, and cache survive because they are stored in the
mounted host directory.

Do not delete `.autograder-cache` if you want to retain the local
database, cached LLM responses, and generated bundles.

------------------------------------------------------------------------

## 10. Rules and problem inputs

`rules.json` defines the problem identifier, resource limits, metrics,
weights, and analyzer parameters. The exact schema is validated by
AutoGrader.

A typical problem folder may look like:

``` text
problems/
└── add_two_integers/
    ├── rules.json
    ├── problem.md             # optional statement
    ├── reference.c            # optional instructor reference
    ├── tests/                 # optional instructor tests
    ├── edge_cases.json        # optional corner cases
    ├── gen.py                 # optional instructor generator
    ├── submissions/
    └── known/                 # optional comparison corpus
```

For the minimal automatic workflow, the essential inputs are a
submission and valid rules. Additional instructor-provided material
improves the reliability and specificity of the grading process.

Rules should define the problem identifier and the metrics to run.
Metric weights should sum to `1.00`. Use the repository's examples as
schema references and validate rules before grading.

------------------------------------------------------------------------

## 11. CLI reference

The web UI is the recommended workflow for users who want to upload
submissions and inspect results in a browser. The CLI is available for
automation and development.

When using the repository's Docker wrapper:

``` bash
./grade.sh <command>
```

On Windows, use the PowerShell wrapper if present:

``` powershell
.\grade.ps1 <command>
```

The main commands are:

  ----------------------------------------------------------------------------
  Command                                  Purpose
  ---------------------------------------- -----------------------------------
  `grade SUB.c --rules R.json`             Grade one submission

  `batch DIRECTORY --rules R.json`         Grade all C files in a directory

  `verify SUB.c --rules R.json --runs 3`   Repeat grading and compare report
                                           fingerprints

  `prepare --rules R.json`                 Prepare and validate assets without
                                           grading

  `serve`                                  Start the local web UI

  `llm-check`                              Check LLM connectivity and model
                                           availability

  `check-rules R.json`                     Validate a rules file

  `tools`                                  Display installed toolchain
                                           versions and analyzers
  ----------------------------------------------------------------------------

Useful flags include:

-   `--generate-assets`: Prepare missing assets.
-   `--offline`: Do not make LLM calls; use available cache and stored
    bundles.
-   `--out DIR`: Select the report output directory.
-   `--cache DIR`: Select the cache directory.
-   `--keep`: Retain temporary build files where supported.
-   `--quiet`: Suppress the text report in CLI output.

For exact options, run the relevant command with `--help`.

------------------------------------------------------------------------

## 12. Configuration

Common environment variables:

  -----------------------------------------------------------------------
  Variable                            Purpose
  ----------------------------------- -----------------------------------
  `AUTOGRADER_CACHE`                  Cache root; defaults to
                                      `.autograder-cache`

  `OLLAMA_HOST`                       Ollama API address as seen by the
                                      application

  `AUTOGRADER_LLM_PROVIDER`           Selects the asset-generation LLM
                                      provider

  `AUTOGRADER_LLM_MODEL`              Selects the LLM model

  `GROQ_API_KEY`                      API key when using the Groq
                                      provider

  `OPENAI_COMPAT_BASE_URL`            Base URL for an OpenAI-compatible
                                      service

  `OPENAI_COMPAT_API_KEY`             API key for that service

  `AUTOGRADER_PORT`                   Host port for the web UI when using
                                      the wrapper
  -----------------------------------------------------------------------

For Docker, the Ollama URL must be reachable from inside the container.
On Docker Desktop for Windows, use:

``` text
http://host.docker.internal:11434
```

On Linux, add the host-gateway mapping shown in section 5 and configure
Ollama to listen on a reachable host interface.

------------------------------------------------------------------------

## 13. Security and reproducibility

### Running untrusted submissions

AutoGrader compiles and executes student code. Use the Docker
environment and its resource limits for untrusted submissions. Do not
expose the grading web server to the public internet. The documented
local setup binds the UI to the local machine.

Treat uploaded C files, rules, and generated assets as untrusted input.
Only load trusted custom Python analyzer plugins. Do not run the grader
with elevated host privileges unless required for Docker administration.

### LLM-generated content

LLM-generated specifications, tests, and oracle programs can be
incomplete or incorrect. Validation reduces risk but does not eliminate
it. Review results before relying on them for consequential decisions.

### Reproducibility

AutoGrader records a report fingerprint intended to identify
grading-relevant results. The Docker image pins important tool versions.
Reproducibility can still depend on platform architecture,
compiler/toolchain versions, and external model behavior. Compare tool
versions and report metadata when investigating differences across
machines.

------------------------------------------------------------------------

## 14. Troubleshooting

  -----------------------------------------------------------------------
  Problem                             What to check
  ----------------------------------- -----------------------------------
  `docker` is not recognized          Install Docker Desktop or Docker
                                      Engine, start it, and reopen the
                                      terminal

  Docker daemon is unavailable        Start Docker Desktop on Windows or
                                      the Docker service on Linux

  `docker build` cannot find the      Run the command from the repository
  Dockerfile                          root

  `localhost:8000` does not open      Check that the container is still
                                      running and inspect its terminal
                                      output

  Port 8000 is already in use         Publish another host port, for
                                      example `-p 9000:8000`, then open
                                      `http://localhost:9000`

  Docker cannot reach Ollama          Check that Ollama is running, the
                                      model is pulled, and `OLLAMA_HOST`
                                      points to the host-reachable
                                      address

  Ollama returns model not found      Run `ollama pull qwen2.5:7b` on the
                                      machine hosting Ollama

  LLM generation is unavailable       Check the Ollama API,
                                      provider/model settings, and the
                                      asset-generation log

  Logical tests are not performed     Inspect the Assets tab and report
                                      for missing or rejected expected
                                      outputs, oracle disagreements, and
                                      validation reasons

  Complexity is not performed         Check whether a valid input
                                      specification and generator were
                                      produced and whether requested
                                      sizes are within the inferred input
                                      limits

  Generated assets are not visible    Check
                                      `.autograder-cache/generated/` in
                                      the host project directory and
                                      inspect the Assets tab

  Reports are missing                 Check `reports/` and confirm the
                                      repository is mounted into `/work`

  Permission denied on Linux          Check ownership and write
                                      permissions for the repository,
                                      `reports/`, and
                                      `.autograder-cache/`

  Similarity is marked suspect        Review fingerprint counts and
                                      comparison evidence; small or
                                      common programs may not provide
                                      enough distinctive evidence for a
                                      confident flag
  -----------------------------------------------------------------------

### Useful Docker diagnostics

List running containers:

``` bash
docker ps
```

List all containers, including stopped ones:

``` bash
docker ps -a
```

View logs:

``` bash
docker logs <container_id>
```

Check images:

``` bash
docker images
```

Check the Ollama API from the host:

``` bash
curl http://localhost:11434/api/tags
```

------------------------------------------------------------------------

## 15. Development and tests

For development outside Docker, the repository README describes a Python
installation path. A typical Linux setup is:

``` bash
python3 -m venv .venv
source .venv/bin/activate
pip install -e '.[dev]'
autograder tools
python -m pytest -q
```

On Windows PowerShell, activate the virtual environment with:

``` powershell
py -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -e ".[dev]"
autograder tools
python -m pytest -q
```

For the Docker test suite, the repository provides:

``` bash
./scripts/test_in_docker.sh
```

Use the tests to check changes to the grading pipeline, asset
preparation, analyzers, web API, and reporting behavior.

------------------------------------------------------------------------

## Further documentation

-   [`docs/DESIGN.md`](docs/DESIGN.md) --- analyzer methods, design
    decisions, usefulness, and limitations.
-   [`autograder/assets/prepare.py`](autograder/assets/prepare.py) ---
    asset preparation pipeline.
-   [`autograder/assets/oracle.py`](autograder/assets/oracle.py) ---
    oracle validation and expected-output trust policy.
-   [`autograder/sandbox.py`](autograder/sandbox.py) --- execution
    sandbox and resource limits.
-   [`tests/`](tests/) --- automated tests.

## License

Add the repository's applicable license information here. Do not assume
a license unless one is present in the repository.
