The command line
==================

::

    pbip-forge check SPEC [--profile {mockup,handoff}] [--format {text,json}]
    pbip-forge build SPEC OUT_DIR [--check] [--profile {mockup,handoff}] [--no-handoff]
    pbip-forge rules
    pbip-forge table dump FILE.tmdl [--max-rows N]
    pbip-forge table dump --b64 STRING [--max-rows N]
    pbip-forge report blank --client DIR --model NAME [NAME ...] [--name REPORT]
                            [--theme NAME] [--theme-file PATH] [--full] [--dry-run]
    pbip-forge clone-report TEMPLATE [NAME] [TARGET_DIR] [SEMANTIC_MODEL]
                            [--model MODEL] [--force] [--logical-id UUID] [--strict]
                            [--expect-visuals N] [--expect-bookmark NAME]
    pbip-forge f-filters (--model M.SemanticModel | --fleet DIR) [--dry-run]
                         [--no-backups] [-v]
    pbip-forge i-filters (--model M.SemanticModel | --fleet DIR) [--dry-run]
                         [--no-backups] [-v] [--skip-validation]
    pbip-forge --version | --help

``pbip-forge`` 1.2.1 has eight commands. Three work on a spec, the object
that describes a model (:doc:`spec-format`). Five work on a PBIP project
that already exists on disk; they have their own page,
:doc:`project-commands`.

.. list-table::
   :widths: 20 80
   :header-rows: 1

   * - Command
     - What it does
   * - ``check``
     - Judges a spec against the 22 modeling rules, before any file is
       written. See `check`_ and :doc:`check-rules`.
   * - ``build``
     - Writes the whole PBIP from a spec: the ``.pbip`` file, the semantic
       model, a report with one empty page, and the hand-off pack. See
       `build`_ and :doc:`handoff-pack`.
   * - ``rules``
     - Prints the rule catalog. See `rules`_.
   * - ``table dump``
     - Prints the rows of an encoded "Enter data" table.
   * - ``report blank``
     - Creates a report with one empty page, bound to a model that already
       exists.
   * - ``clone-report``
     - Copies a report folder under a new name and a new identity.
   * - ``f-filters``
     - Fills the ``f-<Category>`` measures and the ``f-All`` gate lines of
       a model built on the selector pattern.
   * - ``i-filters``
     - Regenerates the ``i-Applied Filters`` caption measure.

The transcripts on these pages are real runs of ``pbip-forge`` 1.2.1 and
``tmdl-preflight`` 1.2.1, made from the root of the tool's repository
(:doc:`../how-to/install` shows how to clone it).

check
------

::

    pbip-forge check SPEC [--profile {mockup,handoff}] [--format {text,json}]

``SPEC`` is a ``.json`` file holding the spec, or a ``.py`` file that
defines a module-level ``SPEC`` dict, like the example scripts in the
tool's repository.

.. list-table::
   :widths: 22 14 64
   :header-rows: 1

   * - Option
     - Default
     - Meaning
   * - ``--profile``
     - ``mockup``
     - ``mockup`` or ``handoff``. The profile decides which documentation
       and naming rules are P1 and which are advisory P2. See
       :doc:`check-rules`.
   * - ``--format``
     - ``text``
     - ``text`` prints one line per violation and a summary; ``json``
       prints the same findings as a JSON object.

A spec that passes prints only the summary line:

::

    pbip-forge check scripts/build_vet_clinic.py --profile handoff
    pbip-forge check (handoff): 0 P0, 0 P1, 0 P2

A violation is printed with its fix indented below it. This run is the
hospital example with ``Patients #`` changed to count the ``Patients``
table, ``DISTINCTCOUNT(Patients[patient_id])``:

::

    P1 FM015 'Clinic Measures'[Avg Billed per Patient]: DIVIDE denominator counts dimension table 'Patients', so filters from other dimensions do not reach it
        fix: Count from the fact instead: DISTINCTCOUNT(Encounters[patient_id]).
    pbip-forge check (handoff): 0 P0, 1 P1, 0 P2

The same run with ``--format json``:

::

    {
      "profile": "handoff",
      "violations": [
        {
          "rule": "FM015",
          "severity": "P1",
          "object": "'Clinic Measures'[Avg Billed per Patient]",
          "message": "DIVIDE denominator counts dimension table 'Patients', so filters from other dimensions do not reach it",
          "fix": "Count from the fact instead: DISTINCTCOUNT(Encounters[patient_id])."
        }
      ]
    }

The command exits 1 when it finds any P0 or P1 violation and 0
otherwise, in both formats. A spec it cannot load also exits 1: a file
that is neither ``.json`` nor ``.py`` prints ``<path>: expected a .json
or .py spec file``, and a ``.py`` file without ``SPEC`` prints ``<path>
does not define a module-level SPEC dict``. A ``.json`` path that does
not exist ends in a Python traceback (``FileNotFoundError``), also with
exit 1. A usage error, such as a missing spec or an unknown option,
exits 2.

build
------

::

    pbip-forge build SPEC OUT_DIR [--check] [--profile {mockup,handoff}] [--no-handoff]

Writes the model into ``OUT_DIR``. The spec is read as ``check`` reads it;
the spec comes first and the output folder second.

.. list-table::
   :widths: 22 14 64
   :header-rows: 1

   * - Option
     - Default
     - Meaning
   * - ``--check``
     - off
     - Run ``check`` first. On a P0 or P1 violation, print the check report,
       write nothing and exit 1.
   * - ``--profile``
     - ``mockup``
     - The profile ``--check`` runs under.
   * - ``--no-handoff``
     - off
     - Do not write ``handoff/manifest.json`` and ``handoff/SPEC.md``.

A successful build prints one line and exits 0:

::

    pbip-forge build scripts/build_vet_clinic.py out/VetClinic --check --profile handoff
    Built PBIP: out/VetClinic/VetClinic.pbip

It writes this tree:

::

    out/VetClinic/
        VetClinic.pbip
        VetClinic.SemanticModel/
            .platform
            definition.pbism
            definition/
                database.tmdl
                model.tmdl
                relationships.tmdl
                tables/<one .tmdl file per table>
        VetClinic.Report/
            .platform
            definition.pbir
            definition/
                report.json
                version.json
                bookmarks/
                pages/
                    pages.json
                    <page id>/page.json
            StaticResources/
        handoff/
            SPEC.md
            manifest.json

The report is a complete PBIR report with one empty page, ``Page 1``
(1280 x 720), and no visuals: the same structure ``report blank`` writes.
Every id in the tree, the report's ``logicalId`` and page id included, is
derived from the model's names, so building the same spec twice gives
identical files.

A build into an existing folder deletes every ``.tmdl`` file in
``definition/tables`` first, then rewrites every file in the tree above.
Other files in the folder are left alone.

With ``--check``, a spec that fails writes nothing and exits 1, after
printing the same report ``check`` prints.

``build`` does not validate column types or table names beyond what
``check`` covers. A column type outside the supported list, or a table
named ``Measures``, passes ``check`` and stops ``build`` with a Python
traceback ending in a ``ValueError`` that names the problem; the run
exits 1. :doc:`spec-format` has both messages.

From Python, the same build is ``build_pbip(spec, out_dir, check=True,
profile="handoff")``; it raises ``SpecCheckError`` without writing
anything when the check fails.

rules
------

::

    pbip-forge rules

Prints the 22 rules, each with its name, its severity in both profiles,
why it exists and how to fix it. It takes no options and exits 0. The
first two entries:

::

    FM001  relationship-endpoints     mockup P0  handoff P0
          A relationship whose table or column does not exist cannot be built.
          fix: Correct the table or column name in the relationship, or add the column.

    FM002  row-width                  mockup P0  handoff P0
          Every sample row must have one value per declared column, or the entered-data partition loads shifted or missing values.
          fix: Add or remove values so each row matches the column list.

:doc:`check-rules` has the whole catalog.

The two gates
--------------

A model is ready to hand off when it passes both gates, in this order:

#. ``pbip-forge check SPEC --profile handoff`` reports 0 P0 and 0 P1.
#. ``tmdl-preflight check OUT_DIR`` reports 0 errors.

For the veterinary clinic example, from the root of the tool's
repository:

::

    pbip-forge check scripts/build_vet_clinic.py --profile handoff
    pbip-forge check (handoff): 0 P0, 0 P1, 0 P2

    pbip-forge build scripts/build_vet_clinic.py out/VetClinic --check --profile handoff
    Built PBIP: out/VetClinic/VetClinic.pbip

    tmdl-preflight check out/VetClinic
    VetClinic.SemanticModel/definition/tables/__Calendar.tmdl  R013 info: calendar __Calendar has no Year, Month, Week, Day column(s). [__Calendar]

    scanned: 1 model, 7 tables, 38 columns, 11 measures, 6 relationships, 12 DAX expressions, 1 report
    ...
    tmdl-preflight: 0 error(s), 0 warning(s), 1 info(s)

The ``...`` stands for the per-rule summaries tmdl-preflight prints
between those lines. The whole project passes, the report included:
since 1.2.1 the report ``build`` writes has the ``report.json``,
``version.json`` and ``pages.json`` that tmdl-preflight's report rules
look for. The R013 info is advisory; the generated calendar names its
columns ``Calendar Year``, ``Calendar Month`` and so on.

Passing both gates proves the files pass both tools' rules. It does not
prove that Power BI Desktop opens the model: open it yourself, on
Windows, to know that.

Exit codes
-----------

.. list-table::
   :widths: 14 86
   :header-rows: 1

   * - Exit code
     - When
   * - 0
     - The command did what it was asked. For ``check`` and ``build
       --check``: no P0 or P1 violation.
   * - 1
     - ``check`` found a P0 or P1 violation; a spec, file, model or
       template was missing or could not be read; a report or a model
       failed; or ``build`` stopped on an error.
   * - 2
     - A usage error: a missing command or argument, an unknown option or
       choice, ``--model`` together with ``--fleet``, or ``--name`` with
       more than one ``--model``. ``clone-report`` also exits 2 for a
       target inside the template or a ``--logical-id`` that is not a
       UUID.

:doc:`project-commands` gives each project command's exit codes in
detail.
