# ACE-Step 1.5 — RX 9070 XT, Linux x86_64, ROCm 7.2

This is the setup used to run ACE-Step on an AMD Radeon RX 9070 XT with
NodePhell. The web interface starts and generates music successfully.

## Get it running

Start with [ACE-Step 1.5](https://github.com/ace-step/ACE-Step-1.5/tree/ca1e85fe9430179831e6bc6be790c332190a3866)
and [NodePhell installed from its source checkout](https://github.com/JustTryingToGetSomeWorkDone/NodePhell#getting-started).
You will also need working AMD GPU drivers for your system.

Copy these three files into your ACE-Step folder, beside its `pyproject.toml`:

- `requirements-rocm72-linux-complete.txt`
- `nodephell.lock.toml`
- `start_gradio_ui_rocm.sh` — replace the upstream launcher with this copy.

Keep ACE-Step's own `pyproject.toml` and `requirements-rocm-linux.txt`. Our
complete profile includes that requirements file and adds the matched ROCm
dependencies. The supplied launcher fixes two problems with the upstream
script, described below.

Open a terminal in your ACE-Step folder and run:

```sh
nodephell sync --requirements requirements-rocm72-linux-complete.txt
chmod +x start_gradio_ui_rocm.sh
./start_gradio_ui_rocm.sh
```

NodePhell installs what is missing and reuses packages you already have.
There is no uv or virtual environment setup to do. The first run also
downloads ACE-Step's models into `checkpoints/`, so leave room for both the
Python packages and models before starting.

Open the address printed by the launcher, normally `http://127.0.0.1:7860`.
Use the `pt` language-model backend and leave int8 disabled for the recorded
workflow. Once the models are ready, you can generate music.

For later launches, just run `./start_gradio_ui_rocm.sh` again. If you need
to resynchronize, `nodephell sync` remembers the selected profile.

### A couple of launcher details

The launcher may say `venv_rocm not found` and `Using system Python`. In this
setup it runs `python3` from PATH, which should be NodePhell's shim. You can
check it from the ACE-Step folder with:

```sh
command -v python3
python3 -c 'import sys, torch; print(sys.executable); print(torch.__version__); print("ROCm:", torch.version.hip)'
```

The command should select your NodePhell shim, and the interpreter path should
be under `~/.python/`. The lock selects torch `2.11.0+rocm7.2`.
Use a terminal without an activated environment; the upstream launcher's
existing `venv_rocm` activation branch is retained if that directory exists.

The script checks for upstream updates. Decline an update when reproducing
the tested revision, or skip that check with:

```sh
CHECK_UPDATE=false ./start_gradio_ui_rocm.sh
```

## Recorded validation

- **Status:** user-confirmed successful web UI startup and music generation.
- **Test date:** 2026-11-18, as recorded in the supplied configuration.
- **Upstream revision:** `ca1e85fe9430179831e6bc6be790c332190a3866`.
- **Project version:** 1.5.0.
- **Platform and GPU:** Linux x86_64, AMD Radeon RX 9070 XT.
- **Runtime:** managed CPython 3.12.16.
- **Lock:** 122 exact package releases with artifact URLs and SHA-256 hashes.

The supplied lock records these accelerator dependencies:

| Package | Locked version |
| --- | --- |
| torch | 2.11.0+rocm7.2 |
| torchvision | 0.26.0+rocm7.2 |
| torchaudio | 2.11.0+rocm7.2 |
| torchao | 0.17.0+rocm7.2 |
| xformers | 0.0.36 |
| mslk | 1.1.0+rocm7.2.26015 |
| peft | 0.21.3 |

The profile selects xformers 0.0.36 with its separate MSLK native-kernel
dependency for this torch/ROCm release family. nano-vllm is omitted for the
`pt` backend because its upstream Linux metadata selects a CUDA FlashAttention
artifact.

Compared with the upstream launcher at the recorded revision, the supplied
script contains two user-tested fixes:

- It disables the default `HSA_OVERRIDE_GFX_VERSION=11.0.0` assignment, allowing
  the RX 9070 XT to use native GPU detection instead of the RDNA3 override.
- It reads `LANGUAGE` as `${LANGUAGE:-en}`, avoiding an unbound-variable exit
  under `set -u` when that environment variable is absent.

This record covers the user's RX 9070 XT run; optional training and other GPUs
or upstream revisions were not tested here. The supplied profile and lock are
preserved.

The launcher comes from ACE-Step and remains under its MIT license; see
`LICENSE-ACE-Step.txt`. The dependency configuration and original documentation
follow this repository's CC0 terms.
