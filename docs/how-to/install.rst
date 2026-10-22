Install the tools
===================

You need two command-line tools at version 1.2.1: ``pbip-forge``, from
this project, and ``tmdl-preflight``, the separate tool that validates
the files ``pbip-forge`` writes. Install both once. Every command on
these pages is the same in PowerShell and in bash unless the page shows
two forms.

You will need:

- Python 3.10 or later (``tmdl-preflight`` requires 3.10; ``pbip-forge``
  alone runs on 3.9).
- Git.
- Either `uv <https://docs.astral.sh/uv/>`_ or ``pip``.
- Power BI Desktop, on Windows, only to open the result yourself. The
  tools do not need it.

Install with uv
-----------------

::

    uv tool install git+https://github.com/C-Kapsalis/pbip-model-forge@v1.2.1
    uv tool install git+https://github.com/C-Kapsalis/tmdl-preflight@v1.2.1

If uv warns that its tool folder is not on your ``PATH``, run ``uv tool
update-shell`` and open a new terminal.

Install with pip
------------------

Use a virtual environment.

In PowerShell:

::

    python -m venv .venv
    .\.venv\Scripts\Activate.ps1
    python -m pip install "git+https://github.com/C-Kapsalis/pbip-model-forge.git@v1.2.1"
    python -m pip install "git+https://github.com/C-Kapsalis/tmdl-preflight.git@v1.2.1"

In bash:

::

    python3 -m venv .venv
    source .venv/bin/activate
    python -m pip install "git+https://github.com/C-Kapsalis/pbip-model-forge.git@v1.2.1"
    python -m pip install "git+https://github.com/C-Kapsalis/tmdl-preflight.git@v1.2.1"

Check the versions
--------------------

::

    pbip-forge --version
    tmdl-preflight --version

Both must print 1.2.1:

::

    pbip-forge 1.2.1
    tmdl-preflight 1.2.1

Run without installing
------------------------

``uvx`` runs a tool at a pinned version without installing it. Put the
prefix in front of any command on these pages:

::

    uvx --from git+https://github.com/C-Kapsalis/pbip-model-forge@v1.2.1 pbip-forge --version
    uvx --from git+https://github.com/C-Kapsalis/tmdl-preflight@v1.2.1 tmdl-preflight --version

Get the example files
-----------------------

The worked examples (the veterinary clinic and the hospital), the test
fixtures that :doc:`../reference/project-commands` runs on, and the two
Claude Code skills live in the tool's repository. Clone it at the same
tag:

::

    git clone --branch v1.2.1 https://github.com/C-Kapsalis/pbip-model-forge.git
    cd pbip-model-forge

Git reports that the clone is in "detached HEAD" state, which is what a
tag checkout is. You only need the clone for the examples and the
skills; :doc:`build-a-model-from-a-spec` needs nothing but the two
installed tools.
