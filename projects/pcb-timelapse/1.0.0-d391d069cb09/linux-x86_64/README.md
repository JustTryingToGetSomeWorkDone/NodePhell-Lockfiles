# PCB Timelapse 1.0.0 — Linux x86_64

**Status: validated for the recorded KiCad rendering workflow.**

Configuration for [PCB Timelapse](https://github.com/MrJamington/pcb-timelapse/tree/d391d069cb098c235c7d7b1a93e2e92b29af38c6),
version 1.0.0, commit `d391d069cb098c235c7d7b1a93e2e92b29af38c6`.
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

The plugin's render screen had display issues reported upstream. Those did
not prevent renderer startup or dependency selection. Other KiCad versions,
other platforms, and newer PCB Timelapse revisions were not validated by this
record. The files are preserved from the tested 1.0.0 setup even though the
local upstream checkout has subsequently advanced.
