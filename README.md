# AutoGrader: a rule-driven, reproducible grader for C programs

AutoGrader takes a student's `.c` file and an instructor's rules file. It runs
the program safely under time and memory limits and checks it with one analyzer
per metric: correctness, warnings, structure, modularity, corner cases,
complexity, memory safety, readability and originality. It returns **one
weighted grade** plus feedback. For every failure the feedback shows the input
that breaks the program, the expected output beside it, and why.


## 1. Installation and first run

Choose **Docker** for the most reproducible results. The `Dockerfile` is in the
**repository root** (the same directory as this README), not inside the
`autograder/` Python package directory. Run all Docker build commands from that
repository root.

### Prerequisites

- Git, to clone the repository.
- Docker Desktop on Windows/macOS, or Docker Engine on Linux.
- (Optional) Ollama on your host computer if you want the local LLM readability
  metric. The grader can run without Ollama; cached answers can be used offline.

Check that Docker is installed and running:

```text
docker --version
docker info
```

If `docker info` reports that it cannot connect to the daemon, start Docker
Desktop (Windows/macOS) or the Docker service (Linux) and try again.

### Step 1 — Get the repository

Replace `<REPOSITORY_URL>` with the clone URL of your repository:

```sh
git clone <REPOSITORY_URL> autograder-project
cd autograder-project
```

Confirm that you are in the repository root. You should see this README,
`Dockerfile`, `grade.sh`, `pyproject.toml`, `autograder/`, `problems/`, and
`scripts/`.

### Step 2 — Build the Docker image

Run this command **from the repository root**:

```sh
docker build -t autograder:latest .
```

The final `.` is important: it makes the current repository root the build
context, where the `Dockerfile` lives. The image includes the pinned Linux
toolchain used for reproducible grading (including GCC, Valgrind, cppcheck and
lizard).

### Step 3 — Confirm the toolchain

The direct Docker command works on Windows CMD, PowerShell, macOS and Linux:

```sh
docker run --rm -v "<ABSOLUTE_REPOSITORY_PATH>:/work" -w /work autograder:latest tools
```

Use the platform-specific form below instead of typing the placeholder path.

**Windows — PowerShell**

Run from the repository root:

```powershell
docker run --rm -v "${PWD}:/work" -w /work autograder:latest tools
```

**Windows — Command Prompt (CMD)**

Run from the repository root:

```bat
docker run --rm -v "%cd%:/work" -w /work autograder:latest tools
```

**macOS / Linux**

Run from the repository root:

```sh
docker run --rm -v "$PWD:/work" -w /work autograder:latest tools
```

On Windows, Docker Desktop must be running and configured to use Linux
containers. If a bind mount fails, make sure the project is stored in a folder
Docker Desktop can access.

### Step 4 — Run commands and view help

All paths passed to the grader are relative to the repository root mounted at
`/work` inside the container. Run these commands from the folder containing
`Dockerfile`, `grade.sh`, `autograder/`, and `problems/`.

#### Windows — Command Prompt (CMD)

**Show the main help and command-specific help:**

```bat
docker run --rm -v "%cd%:/work" -w /work autograder:latest --help
docker run --rm -v "%cd%:/work" -w /work autograder:latest grade --help
docker run --rm -v "%cd%:/work" -w /work autograder:latest batch --help
docker run --rm -v "%cd%:/work" -w /work autograder:latest verify --help
docker run --rm -v "%cd%:/work" -w /work autograder:latest check-rules --help
docker run --rm -v "%cd%:/work" -w /work autograder:latest tools
```

**Grade one submission** — creates a JSON report and a readable text report:

```bat
docker run --rm -v "%cd%:/work" -w /work autograder:latest grade problems\add_two_integers\submissions\add.c --rules problems\add_two_integers\rules.json
```

**Grade every submission in a folder** — writes per-file reports and a summary CSV:

```bat
docker run --rm -v "%cd%:/work" -w /work autograder:latest batch problems\max_subarray\submissions --rules problems\max_subarray\rules.json --out reports\max_subarray
```

**Check reproducibility** — grade the same submission three times and compare report fingerprints:

```bat
docker run --rm -v "%cd%:/work" -w /work autograder:latest verify problems\max_subarray\submissions\s05_leaky.c --rules problems\max_subarray\rules.json --runs 3
```

**Validate a rules file without grading:**

```bat
docker run --rm -v "%cd%:/work" -w /work autograder:latest check-rules problems\max_subarray\rules.json
```

**Run in offline mode** — use cached LLM responses only, without calling a model:

```bat
docker run --rm -v "%cd%:/work" -w /work autograder:latest batch problems\max_subarray\submissions --rules problems\max_subarray\rules.json --out reports\max_subarray --offline
```

#### Windows — PowerShell

PowerShell uses `${PWD}` for the current directory. Run each command from the
repository root.

**Help and toolchain:**

```powershell
docker run --rm -v "${PWD}:/work" -w /work autograder:latest --help
docker run --rm -v "${PWD}:/work" -w /work autograder:latest grade --help
docker run --rm -v "${PWD}:/work" -w /work autograder:latest batch --help
docker run --rm -v "${PWD}:/work" -w /work autograder:latest verify --help
docker run --rm -v "${PWD}:/work" -w /work autograder:latest check-rules --help
docker run --rm -v "${PWD}:/work" -w /work autograder:latest tools
```

**Grade one submission:**

```powershell
docker run --rm -v "${PWD}:/work" -w /work autograder:latest grade problems/add_two_integers/submissions/add.c --rules problems/add_two_integers/rules.json
```

**Grade a directory:**

```powershell
docker run --rm -v "${PWD}:/work" -w /work autograder:latest batch problems/max_subarray/submissions --rules problems/max_subarray/rules.json --out reports/max_subarray
```

**Verify, validate rules, or run offline:**

```powershell
docker run --rm -v "${PWD}:/work" -w /work autograder:latest verify problems/max_subarray/submissions/s05_leaky.c --rules problems/max_subarray/rules.json --runs 3
docker run --rm -v "${PWD}:/work" -w /work autograder:latest check-rules problems/max_subarray/rules.json
docker run --rm -v "${PWD}:/work" -w /work autograder:latest batch problems/max_subarray/submissions --rules problems/max_subarray/rules.json --out reports/max_subarray --offline
```

#### macOS / Linux

Run from the repository root. These examples use `$PWD` for the current
directory.

**Help and toolchain:**

```sh
docker run --rm -v "$PWD:/work" -w /work autograder:latest --help
docker run --rm -v "$PWD:/work" -w /work autograder:latest grade --help
docker run --rm -v "$PWD:/work" -w /work autograder:latest batch --help
docker run --rm -v "$PWD:/work" -w /work autograder:latest verify --help
docker run --rm -v "$PWD:/work" -w /work autograder:latest check-rules --help
docker run --rm -v "$PWD:/work" -w /work autograder:latest tools
```

**Grade one submission:**

```sh
docker run --rm -v "$PWD:/work" -w /work autograder:latest grade problems/add_two_integers/submissions/add.c --rules problems/add_two_integers/rules.json
```

**Grade a directory:**

```sh
docker run --rm -v "$PWD:/work" -w /work autograder:latest batch problems/max_subarray/submissions --rules problems/max_subarray/rules.json --out reports/max_subarray
```

**Verify, validate rules, or run offline:**

```sh
docker run --rm -v "$PWD:/work" -w /work autograder:latest verify problems/max_subarray/submissions/s05_leaky.c --rules problems/max_subarray/rules.json --runs 3
docker run --rm -v "$PWD:/work" -w /work autograder:latest check-rules problems/max_subarray/rules.json
docker run --rm -v "$PWD:/work" -w /work autograder:latest batch problems/max_subarray/submissions --rules problems/max_subarray/rules.json --out reports/max_subarray --offline
```

#### Using `grade.sh` instead of typing Docker commands

On macOS/Linux, WSL, or Git Bash with Docker Desktop configured, the wrapper
can shorten the commands. Run from the repository root:

```sh
chmod +x grade.sh
./grade.sh --help
./grade.sh grade --help
./grade.sh batch --help
./grade.sh verify --help
./grade.sh check-rules --help
./grade.sh tools

./grade.sh grade problems/add_two_integers/submissions/add.c --rules problems/add_two_integers/rules.json
./grade.sh batch problems/max_subarray/submissions --rules problems/max_subarray/rules.json --out reports/max_subarray
./grade.sh verify problems/max_subarray/submissions/s05_leaky.c --rules problems/max_subarray/rules.json --runs 3
./grade.sh check-rules problems/max_subarray/rules.json
./grade.sh batch problems/max_subarray/submissions --rules problems/max_subarray/rules.json --out reports/max_subarray --offline
```

`grade.sh` is a Bash script and does not run directly in Windows CMD. In CMD
or PowerShell, use the direct Docker commands in the relevant section above.
For WSL/Git Bash, if the script reports `^M`, `CRLF`, or a bad interpreter
error, convert it to LF line endings:

```sh
dos2unix grade.sh
```

Or, if `dos2unix` is unavailable:

```sh
sed -i 's/\r$//' grade.sh
```

#### What the commands do

- `grade FILE --rules RULES`: grade one C submission.
- `batch DIRECTORY --rules RULES`: grade all supported submissions in a folder.
- `verify FILE --rules RULES --runs N`: repeat grading and compare report fingerprints.
- `check-rules RULES`: validate a rules file without grading a submission.
- `tools`: print the toolchain available inside the image.
- `--out DIRECTORY`: choose where reports are written.
- `--offline`: use cached LLM answers only; it does not call a model.
- `--cache DIRECTORY`: choose the LLM answer cache directory.
- `--keep`: preserve the temporary build directory for debugging.

For the exact accepted arguments and defaults in your installed version, run
the command-specific `--help` command shown above. If a particular command
does not support `--help` in your version, use the top-level `--help` output
and the repository's CLI implementation as the source of truth.

The reports are written under `reports/` by default. If Docker reports that a
rules file or submission cannot be found, check that you ran the command from
the repository root and that the example paths exist. If a volume mount fails
on Windows, confirm Docker Desktop is running and that the project directory
is accessible to Docker Desktop.

### Optional — Enable the LLM readability metric with Ollama

Install Ollama on the **host** computer using the official instructions at
<https://ollama.com/download>, then download the configured model:

```sh
ollama pull qwen2.5:7b
```

Make sure Ollama is running and that this model matches the `llm_review`
configuration in the relevant `rules.json`. The Docker wrapper uses
`host.docker.internal` to reach Ollama on the host. The LLM metric is optional;
without a reachable model, use the committed cache where available:

```sh
./grade.sh batch problems/max_subarray/submissions --rules problems/max_subarray/rules.json --out reports/max_subarray --offline
```

For Windows CMD, replace `./grade.sh` with the equivalent direct Docker command:

```bat
docker run --rm -v "%cd%:/work" -w /work autograder:latest batch problems/max_subarray/submissions --rules problems/max_subarray/rules.json --out reports/max_subarray --offline
```

### Option B — Run from Python (development)

Native Python mode is useful for development, but the available checks depend
on your operating system and installed tools. For the most comparable grades,
use Docker.

Requirements: Python supported by the project configuration, a C compiler
(`gcc` or `clang`), and Git. `cppcheck` and `valgrind` are optional but
recommended. Use the Python version declared in `pyproject.toml` if it differs
from the examples below.

**Windows — PowerShell**

```powershell
py -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
python -m pip install -e ".[dev]"
autograder tools
```

If PowerShell blocks activation, you can skip activation and call the virtual
environment directly:

```powershell
.\.venv\Scripts\python.exe -m pip install -e ".[dev]"
.\.venv\Scripts\autograder.exe tools
```

You must also install a Windows-compatible C compiler and any optional tools
you intend to use. Some sandbox/resource-limit features may be unavailable on
native Windows; WSL2 or Docker is recommended for Linux-like execution.

**macOS / Linux**

```sh
python3 -m venv .venv
. .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install -e '.[dev]'
autograder tools
```

On macOS some checks are reduced: Valgrind and LeakSanitizer are generally not
available, and `RLIMIT_AS` is not enforced. Complexity may fall back to CPU
time and reports may flag `nondeterministic_measurement`. Use Docker for grades
you publish.

## 2. Usage

All commands below assume you are in the repository root. In Docker, replace
`./grade.sh` with `docker run --rm -v "$PWD:/work" -w /work autograder:latest`
on macOS/Linux, or use the Windows-specific mount syntax shown above.

### Grade one submission

```sh
./grade.sh grade problems/add_two_integers/submissions/add.c --rules problems/add_two_integers/rules.json
```

### Grade every C file in a directory

```sh
./grade.sh batch problems/max_subarray/submissions --rules problems/max_subarray/rules.json --out reports/max_subarray
```

This writes a report for each submission and a `summary.csv` under the output
directory.

### Verify reproducibility

```sh
./grade.sh verify problems/max_subarray/submissions/s05_leaky.c --rules problems/max_subarray/rules.json --runs 3
```

This grades the same submission multiple times and compares report fingerprints.

### Validate a rules file without grading

```sh
./grade.sh check-rules problems/max_subarray/rules.json
```

### Discover available tools

```sh
./grade.sh tools
```

Without Docker, replace `./grade.sh` with `autograder`. Useful flags include
`--out DIR`, `--offline` (use cached LLM answers only), `--cache DIR`, and
`--keep` (keep the temporary build directory for debugging).

The LLM answers behind `sample_outputs/` are committed in `.autograder-cache/`.
Where the required cache entries exist, `--offline` can reproduce sample grades
without Ollama or a GPU.

## 3. Writing rules

A problem is a folder. Everything the instructor controls lives in it, and none
of it is code:

```
problems/max_subarray/
  rules.json         which metrics, weights, thresholds, limits   <- the rules
  problem.md         statement (given to the LLM explainer)
  reference.c        correct solution: computes every expected output
  tests/NN.in(.out)  hidden tests (.out optional: made from reference.c)
  edge_cases.json    named corner-case families, e.g. "all_negative"
  gen.py             generate(size, rng) -> input, for stress tests and complexity
  submissions/       the class's submissions (also the plagiarism corpus)
  known/             web / LLM-generated solutions to compare against
```

`rules.json` (the PDF's format, plus a few optional keys):

```json
{
  "problem": "add_two_integers",
  "time_limit_s": 1,
  "memory_limit_mb": 64,
  "sample_input": "7 5\n",
  "reference": "reference.c",
  "metrics": [
    { "name": "correctness", "analyzer": "io_tests", "weight": 0.20,
      "params": { "testset_dir": "tests/" } },
    { "name": "complexity", "analyzer": "complexity_estimator", "weight": 0.10,
      "params": { "required": "O(1)", "sizes": [1, 10, 100] } }
  ]
}
```

Paths are relative to the rules file. Weights must sum to 1.00, and
`check-rules` reports any mistake with the metric's name.

| Top-level key | Meaning | Default |
|---|---|---|
| `problem` | Problem id | required |
| `time_limit_s`, `memory_limit_mb` | Per-run CPU-time and memory limits | 1, 256 |
| `sample_input` | Input for the first-pass smoke run | empty |
| `reference` | Correct C solution; expected outputs come from it | none |
| `statement` | Problem text, used by LLM explanations | none |
| `compile` | `{"cc", "flags", "ldflags"}` for the main build | gcc, `-O2 -std=c11`, `-lm` |
| `plugins` | Extra analyzer files to load (see section 5) | `[]` |
| `include_usefulness_limitations` | Append that section to every report | true |

### Analyzers and their params

| Analyzer | Metric it measures | Key params |
|---|---|---|
| `io_tests` | Hidden tests pass | `testset_dir`, `generated` (seeded big tests), `compare` (`trim`/`exact`/`tokens`/`float:1e-6`), `explain` (`llm`) |
| `compiler_warnings` | Clean compile under strict flags | `flags`, `penalty_per_warning` (0.1) |
| `static_analysis` | Function size and complexity, cppcheck, globals, goto | `max_cyclomatic`, `max_function_length`, `cppcheck` |
| `function_split` | Decomposition into helpers | `min_functions`, `max_main_share` (0.8) |
| `edge_case_runner` | Corner cases and seeded random stress vs. the reference | `cases` (list or `"all"`), `random_tests`, `sizes`, `seed`, `explain` |
| `complexity_estimator` | Growth rate vs. the required bound | `required`, `sizes`, `penalty_per_class` (0.5), `timeout_s` |
| `sanitizer_run` | ASan, UBSan, LSan and valgrind over all inputs | `flags`, `valgrind`, `penalty_per_issue` (0.25), `max_input_bytes` |
| `llm_review` | Readability on a fixed 10-point rubric | `provider` (`ollama`/`groq`), `model`, `temperature`, `seed` |
| `similarity_flag` | Copying from classmates, the web or LLMs | `corpus_dir`, `known_dir`, `flag_threshold`, `flag_only` |
| `source_patterns` | Required or forbidden code (e.g. no `gets`) | `forbid`, `require` (regex + message) |

To catch LLM copies, run
`python scripts/generate_known_solutions.py problems/<p> --count 3`. It asks a
local model for solutions and stores them in `known/`. To write `.out` files
from the reference solution, run `python scripts/make_expected.py problems/<p>`.

## 4. How it works

```
submission.c + rules.json
   │
   ▼  1. load rules ── validate weights, analyzer names, params, paths
   ▼  2. first pass ── compile · smoke run on sample input · static sanity
   │       does not compile / does not halt → skip execution metrics (score 0),
   │       still run source-only ones, mark the report PARTIAL
   ▼  3. dispatch ──── one analyzer per metric, in parallel threads
   │                   (shared builds and reference outputs, all cached)
   ▼  4. collate ──── grade = Σ score × weight · counterexamples grouped by cause
   │                   · suggestions ordered by points lost · flags
   ▼  reports/<name>.json  +  reports/<name>.txt   (+ fingerprint)
```

[`docs/DESIGN.md`](docs/DESIGN.md) explains each analyzer's method and why it
was chosen.

## 5. Adding a metric without editing the agent

Write an analyzer file next to your rules and list it under `"plugins"`. A
complete example is [`examples/plugins/line_budget.py`](examples/plugins/line_budget.py):

```python
from autograder.analyzers import Analyzer, AnalyzerResult, register

@register
class LineBudget(Analyzer):
    name = "line_budget"
    def run(self, ctx, params):
        ...
        return AnalyzerResult(score=..., feedback="...", evidence={...})
```

```json
"plugins": ["line_budget.py"],
"metrics": [ { "name": "conciseness", "analyzer": "line_budget", "weight": 0.05,
               "params": { "max_code_lines": 30 } } ]
```

## 6. Tests

```sh
.venv/bin/python -m pytest -q        # 41 tests on the host (1 is Linux-only and skips on macOS)
./scripts/test_in_docker.sh          # same suite inside the Linux image (valgrind, LSan, RLIMIT_AS)
```

## 7. Repository layout

```
autograder/
  cli.py            grade / batch / verify / check-rules / tools
  coordinator.py    the main agent: first pass, dispatch, collation, fingerprint
  rules.py          load + validate the rules file, hash problem assets
  sandbox.py        run untrusted binaries under CPU/memory/output/wall limits
  context.py        per-submission state: builds, runs, reference outputs, generators
  compiler.py       compile + parse gcc/clang diagnostics
  diagnostics.py    parse ASan / UBSan / LSan / valgrind output
  explain.py        failing run -> WHY + FIX (rule-based, optional cached LLM)
  llm.py            Ollama / Groq client with content-addressed answer cache
  ctokens.py        small C lexer (normalisation, readability facts)
  report.py         JSON + student text report
  analyzers/        one file per analyzer (+ base.py: contract and registry)
  data/usefulness_limitations.json
problems/           two example problems with rules, tests, reference, submissions
sample_outputs/     the agent's reports on every sample submission (Docker toolchain)
scripts/            make_expected.py, generate_known_solutions.py, test_in_docker.sh
examples/plugins/   an analyzer added from the rules file only
tests/              pytest suite
docs/DESIGN.md      design, method defence, usefulness & limitations
```
