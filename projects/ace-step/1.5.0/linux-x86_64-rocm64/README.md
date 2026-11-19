# ACE-Step 1.5 — Linux x86_64, ROCm 6.4

This setup runs ACE-Step with ROCm 6.4 through NodePhell. It has been tested
on an AMD Radeon 880M APU: the web interface starts and generates music.

## Files and installation

Start with [ACE-Step 1.5 at the tested revision](https://github.com/ace-step/ACE-Step-1.5/tree/ca1e85fe9430179831e6bc6be790c332190a3866).
Copy `requirements-rocm64-linux-complete.txt` and `nodephell.lock.toml` from
this folder into your ACE-Step folder, beside its `pyproject.toml`. Keep
ACE-Step's existing files: our profile includes its `requirements-rocm-linux.txt`.

With NodePhell installed from its source checkout, open a terminal in the
ACE-Step folder and run:

```sh
df -h . "$HOME/.python" /tmp
nodephell sync --requirements requirements-rocm64-linux-complete.txt
```

The supplied lock records the selected requirements profile. Synchronization
uses its 121 pinned releases and managed CPython 3.12.15 when the lock matches,
reusing installed artifacts. Subsequent runs can use plain `nodephell sync`;
the selected profile persists. Use NodePhell commit `4ee6c59` or a
later version containing its requirements-profile and missing-hash support.

Allow at least 30 GB of free installation space before the initial sync, plus
additional space for model downloads. The 4.47 GB torch wheel expands to
14.42 GB, and pip can also hold a download and a cached copy during installation.

This replaces the upstream Python environment setup steps: skip `uv sync`,
`uv run`, virtual-environment creation/activation, and separate pip installs
of PyTorch or requirements. Keep using NodePhell for Python dependencies.
GPU drivers and other operating-system prerequisites still need to be present.

To deliberately regenerate the lock from the profile, run:

```sh
nodephell lock -r requirements-rocm64-linux-complete.txt
```

Regeneration may select newer versions of dependencies that the profile leaves
unpinned. Preserve the supplied lock when reproducing this recorded selection.

## Hardware and application setup

An AMD GPU and compatible operating-system GPU drivers/runtime are external
prerequisites. See the upstream
[Linux ROCm instructions](https://github.com/ace-step/ACE-Step-1.5/blob/ca1e85fe9430179831e6bc6be790c332190a3866/docs/en/ACE-Step1.5-Rocm-Manual-Linux.md)
for hardware and application settings; follow the NodePhell setup above for
Python dependencies. The upstream ROCm settings recommend disabling int8 and
selecting the `pt` backend for the 5Hz language model.

The profile consolidates the guide's separate PyTorch installation and Linux
requirements commands into one input. It pins the matched torch, torchvision,
and torchaudio ROCm 6.4 release family, pins `torchao==0.15.0` to match torch
2.9.1, and selects xformers by exact ROCm wheel URL. The torchao pin constrains
the unversioned entry in the included upstream requirements; pip combines
requirements rather than letting the last line override earlier entries.
The lock selects `torchao==0.15.0+rocm6.4`, which satisfies that public-version
pin. nano-vllm is omitted for this `pt` profile because its local Linux metadata
selects a CUDA FlashAttention wheel.

After synchronization succeeds, confirm that `python3` selects NodePhell and
that the matched dependencies import:

```sh
command -v python3
python3 -c 'import sys, torch, torchao, peft; print(sys.executable); print(torch.__version__, torchao.__version__, peft.__version__); print("ROCm:", torch.version.hip)'
```

The command path should identify your NodePhell shim, commonly
`<user-home>/.local/bin/python3`. The interpreter should be under
`<user-home>/.python/`, with torch `2.9.1+rocm6.4`, torchao
`0.15.0+rocm6.4`, PEFT `0.21.3`, and a nonempty ROCm version.

Start the application using the Linux ROCm launcher, as in the recorded run:

```sh
./start_gradio_ui_rocm.sh
```

If its update check offers an upstream update, decline it when reproducing the
revision recorded here. The launcher may print `venv_rocm not found` followed
by `Using system Python`. That is expected for this workflow: without a venv,
this script invokes `python3` from PATH, so the verified NodePhell shim selects
the locked interpreter and packages. No venv needs to be created to silence
the message. An existing `venv_rocm` would instead be activated by the script;
ensure the launcher is using the NodePhell path before interpreting results.

Open the URL printed by the launcher, `http://127.0.0.1:7860` with its default
settings. ACE-Step downloads missing model weights and tokenizers into
`checkpoints/` during first startup. These are application data, separate from
NodePhell's Python package synchronization. Concurrent model downloads may
produce overlapping progress bars in the terminal.

Apply the ROCm settings above, initialize the models, and test an actual
generation. This workflow has been validated through successful web UI startup
and completed music generation on the recorded hardware.

## Recorded results

- **Status:** validated web UI startup and music generation.
- **Upstream version and revision:** ACE-Step 1.5.0,
  `ca1e85fe9430179831e6bc6be790c332190a3866`.
- **GPU:** AMD Radeon 880M APU.

On 2026-11-17, NodePhell commit
`4ee6c598a3c3ae9a657f7d9b59219fa6ec2bb40a` was exercised on Linux Mint 22.3,
x86_64, using managed CPython 3.12.15:

- Resolution produced a 121-package lock with exact artifact URLs and SHA-256.
- Missing SHA-256 values for pytorch-triton-rocm and Jinja2 were recovered by
  downloading the exact artifacts through stock pip and hashing them locally.
- The store reused 24 releases and scheduled 97 installations; 77 completed
  before torch installation failed.
- The 4.473 GB ROCm torch wheel downloaded completely, but extraction failed
  with `OSError: [Errno 28] No space left on device`. Allow space for the
  compressed wheel, temporary extraction, installed libraries, pip cache, and
  models; the wheel size alone is not a sufficient free-space estimate.
- NodePhell cleaned the failed atomic staging directory and preserved the
  complete lock. No partial torch release was committed.
- NodePhell's 203 unit tests passed. Application startup, GPU operation, and
  inference were not tested.

On 2026-11-18, the user completed synchronization after freeing disk space.
Startup detected an AMD Radeon 880M and began downloading model files, but
the unpinned torchao 0.18.0 failed to import with torch 2.9.1. That also caused
PEFT imports to fail through Transformers. Pinning torchao to 0.15.0 installed
one release, reusing the existing torch installation. The user then verified
successful joint imports of torch 2.9.1+rocm6.4, torchao 0.15.0+rocm6.4, and
PEFT 0.21.3. AMD device-ID data and a torchao docstring escape warning remained;
neither prevented those imports.

Following the dependency correction, the user confirmed that manually
launching `./start_gradio_ui_rocm.sh` brought up the web interface and generated
music successfully, without uv or a virtual environment.

The profile now carries the pin, leaving upstream requirements unchanged.
The lock preserves the resolved selection from that successful import check,
with its source fingerprint refreshed for the consolidated profile.
No personal absolute paths are required. Validation covers this revision,
hardware, and generation workflow. Optional training, other backends, other
GPUs, and other upstream revisions have not been validated by this record.
