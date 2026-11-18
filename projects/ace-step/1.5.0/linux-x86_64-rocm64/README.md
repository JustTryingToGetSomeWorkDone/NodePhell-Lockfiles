# ACE-Step 1.5 — Linux x86_64, ROCm 6.4

**Status: candidate; installation and dependency imports verified, generation validation pending.**

This entry preserves a resolved configuration for
[ACE-Step 1.5](https://github.com/ace-step/ACE-Step-1.5/tree/ca1e85fe9430179831e6bc6be790c332190a3866),
commit `ca1e85fe9430179831e6bc6be790c332190a3866` (project version 1.5.0).
It is not yet a validated working application configuration under this
repository's contribution criteria. Installation stopped because the test
machine ran out of disk space. A subsequent run completed synchronization and
verified the dependency imports; inference remains untested.

## Files and installation

Use the upstream checkout at the exact revision above. Keep its existing
`pyproject.toml` and `requirements-rocm-linux.txt`; this entry does not replace
either file. Copy the supplied `requirements-rocm64-linux-complete.txt` and
`nodephell.lock.toml` into the checkout root, beside those upstream files.
The requirements profile includes the upstream Linux ROCm requirements, so it
is not a standalone file for an empty directory.

From that checkout root, check available space and synchronize:

```sh
df -h . "$HOME/.python" /tmp
nodephell sync
```

The supplied lock records the selected requirements profile, so `sync` should
reuse its 121 pinned releases and managed CPython 3.12.15 rather than resolve
the project’s default CUDA dependencies. Use NodePhell commit `4ee6c59` or a
later version containing its requirements-profile and missing-hash support.

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
for hardware and application settings. Those instructions recommend disabling
int8 and selecting the `pt` backend for the 5Hz language model.

The profile consolidates the guide's separate PyTorch installation and Linux
requirements commands into one input. It pins the matched torch, torchvision,
and torchaudio ROCm 6.4 release family, pins `torchao==0.15.0` to match torch
2.9.1, and selects xformers by exact ROCm wheel URL. The torchao pin constrains
the unversioned entry in the included upstream requirements; pip combines
requirements rather than letting the last line override earlier entries.
The lock selects `torchao==0.15.0+rocm6.4`, which satisfies that public-version
pin. nano-vllm is omitted for this `pt` profile because its local Linux metadata
selects a CUDA FlashAttention wheel.

After synchronization succeeds, the upstream local launch command is:

```sh
python -m acestep.acestep_v15_pipeline --server-name 127.0.0.1 --port 7680
```

Open `http://127.0.0.1:7680`, apply the upstream ROCm settings, initialize the
models, and test an actual generation. Model downloads require additional disk
space. These runtime steps have not yet been completed for this entry.

## Recorded results

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

The profile now carries the pin, leaving upstream requirements unchanged.
The lock preserves the resolved selection from that successful import check,
with its source fingerprint refreshed for the consolidated profile.
No personal absolute paths are required. This candidate should be marked
validated only after generation succeeds and its results are recorded.
