# NodePhell-Lockfiles

**Solve a dependency problem once, solve it for everyone.**

Getting Python software running shouldn't be harder than using the software itself. Yet here we are.

NodePhell-Lockfiles is a community repository of validated dependency configurations and lockfiles for Python projects with missing, incomplete, or complex dependency declarations.

The idea is simple. If one of us has already figured out how to get something working, why should everyone else have to figure it out again?

## What is NodePhell?

[NodePhell](https://github.com/JustTryingToGetSomeWorkDone/NodePhell) is an open-source Python dependency and interpreter manager for Linux, built around the idea that managing Python environments shouldn't be a hobby in itself. We have plenty of other things we'd rather be doing.

NodePhell downloads the versions of Python and its packages that your projects need, keeps them on your computer, and reuses them whenever another project needs them. A lockfile tells NodePhell exactly what a project needs to run.

- No virtual environments.
- No activation or deactivation.
- No manual pip installations.
- No worrying about which version of Python is installed or on your PATH.
- No downloading the same packages over and over again.
- The same dependencies every time you run your project.

Just run Python.

Not using NodePhell? If you find them useful, feel free to make use of the dependency declarations here anyway.

## Why this repository exists

Python projects describe their dependencies in all sorts of ways. Some have a well-written `pyproject.toml`, some have requirements files, some have several of each, and some have a README full of commands that must be run in a particular order. Others barely mention their dependencies at all.

Throw in different operating systems, GPU architectures, Python versions, and package indexes, and things can get messy rather quickly.

Even when the installation instructions work, thousands of people may end up repeating the same process to get the same piece of software running.

That's a lot of unnecessary work.

This repository gives us somewhere to share working configurations so that the next person can spend their time using the software instead of figuring out how to install it.

Of course, the best place for these files is in the upstream project itself. More on that below.

## Finding your project

The repository is organized by project name, version, and use case. Some projects may have several configurations depending on your operating system or hardware, while others might only need one.

For example, a project that supports both AMD and NVIDIA GPUs might look something like this:

```text
projects/
└── project-name/
    └── version/
        ├── linux-x86_64-rocm64/
        │   ├── README.md
        │   ├── pyproject.toml
        │   └── nodephell.lock.toml
        └── linux-x86_64-cuda128/
            ├── README.md
            ├── pyproject.toml
            └── nodephell.lock.toml
```

The `pyproject.toml` describes what the project needs. The lockfile contains the exact versions that have been resolved and tested together.

We prefer simple, human-readable `pyproject.toml` files. There's no need to list every dependency of every package. NodePhell works those out when generating the lockfile.

If the original project already provides a suitable `pyproject.toml`, there's no need for us to provide another one. Likewise, if a lockfile is missing, NodePhell can generate one from the dependency declarations.

Where additional information is needed, you'll find it in the accompanying README.

Hopefully, someone has already worked out a configuration that suits your needs!

## Getting started

Find a configuration that matches your project and copy its supplied files into your project directory.

With NodePhell installed, open a terminal in your project directory and run:

```bash
nodephell sync
```

Then run the application as you normally would. For example, if the project's starting script is `main.py`:

```bash
python main.py
```

That's it. NodePhell will take care of the Python version and dependencies, use what you already have, and download whatever is missing.

And the next time you want to run the application? Just run it. There's nothing to activate.

If you're new to NodePhell, have a look at the [documentation](https://github.com/JustTryingToGetSomeWorkDone/NodePhell/tree/main/docs).

## Contributing

Got something working with NodePhell? Great! Share your configuration here so the rest of us don't have to work it out again.

You don't need to know anything about NodePhell's internals, and you certainly don't need to modify NodePhell itself.

All we really need is:

- Your working dependency declaration and lockfile.
- The project version or commit it was made
