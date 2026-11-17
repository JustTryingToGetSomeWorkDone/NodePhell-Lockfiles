# PCB Timelapse — Linux x86_64

**Status: validated for the recorded KiCad rendering workflow.**

Configuration for [PCB Timelapse](https://github.com/MrJamington/pcb-timelapse/tree/d391d069cb098c235c7d7b1a93e2e92b29af38c6),
initially tested with version 1.0.0, commit
`d391d069cb098c235c7d7b1a93e2e92b29af38c6`, and subsequently validated with
[version 1.3.0](https://github.com/MrJamington/pcb-timelapse/tree/ea33c06fbe86d4d3dd1ff780fd65ab6990f54e5d),
commit `ea33c06fbe86d4d3dd1ff780fd65ab6990f54e5d`.
This entry contains only the NodePhell dependency declaration, generated lock,
and setup instructions. Obtain the plugin itself from upstream.

## Setup

Install the KiCad plugin using the upstream instructions. Copy the supplied
`pyproject.toml` and `nodephell.lock.toml` into the plugin directory beside
`render.py`. In a source checkout this is the `pcb_timelapse/` directory,
not the repository root. If KiCad uses a separate installed copy, put the two
files beside that copy's `render.py` as well.

Open a terminal in that directory and run:

```sh
nodephell sync
python render.py --help
```

Synchronization uses the locked CPython 3.12.15 interpreter and package set.
The help command should exit successfully and print renderer arguments.
The supplied project metadata is dependency-only; it does not build or
install the KiCad plugin.

In the plugin's render dialog, set the Python executable to `python` if
KiCad's inherited PATH selects the NodePhell shim. Alternatively, enter the
absolute path to your NodePhell Python shim, commonly
`<user-home>/.local/bin/python`. Expand `<user-home>` to your actual home
directory rather than entering that placeholder literally.

KiCad's embedded Python runs the plugin. The dialog launches `render.py`
through the selected external Python executable, and NodePhell discovers the
metadata beside that script. KiCad itself remains an external prerequisite;
this configuration manages the renderer's Python dependencies.

## Recorded validation

On 2026-10-26 this configuration was tested on Linux Mint 22.3, x86_64,
with KiCad 10.0.7 (`10.0.7~ubuntu24.04.1`) and NodePhell commit
`158ad9459d98ebbf267ea10d2dc6c82be0a3db52`:

- Interactive `nodephell init` generated the dependency declaration and lock.
- Synchronization selected CPython 3.12.15 and 13 package releases, including
  PyMuPDF 2.0 and Pillow 13.0.0.
- `render.py --help` succeeded when launched from outside the project, using
  both the PATH-selected Python shim and its explicit absolute path.
- The KiCad render dialog launched the external renderer and the rendering
  workflow ran with both executable settings.

The 1.0.0 plugin's render screen had display issues reported upstream. Those
did not prevent renderer startup or dependency selection.

On 2026-11-17, upstream was updated to 1.3.0 at the commit recorded above.
All six plugin Python files passed syntax checks, and the renderer still
declared the same PyMuPDF and Pillow dependencies. The user confirmed that
`render.py --help` exited successfully through the NodePhell shim from outside
the project and that rendering through the previously described KiCad
workflow completed successfully with no issues.

The same 13-package selection, artifact hashes, and CPython 3.12.15 interpreter
were retained for both tested versions. Only the lock's source fingerprint was
refreshed for the current NodePhell fingerprint format, avoiding an unnecessary
resolution when using this configuration with current NodePhell.

The `1.0.0` directory names the earliest validated release for this shared
configuration. Validation covers the two revisions listed above; intervening
releases, other KiCad versions, and other platforms have not been tested.
