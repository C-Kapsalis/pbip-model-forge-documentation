The command line
==================

::

    pbip-forge check SPEC [--profile {mockup,handoff}] [--format {text,json}]
    pbip-forge build SPEC OUT_DIR [--check] [--profile {mockup,handoff}]
                            [--no-handoff]
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

``pbip-forge`` has eight commands. Three of them work on a spec — the dict
that describes a model — and five of them work on a PBIP project that
already exists on disk.

.. list-table::
   :widths: 20 80
   :header-rows: 1

   * - Command
     - What it does
   * - ``check``
     - Judges a spec against the 22 modeling rules, before a single file is
       written. See `check`_ and :doc:`check-rules`.
   * - ``build``
     - Writes the whole model — the ``.pbip`` file, the semantic model, the
       blank report folder and the hand-off pack — from a spec. See `build`_
       and :doc:`handoff-pack`.
   * - ``rules``
     - Prints the rule catalog: every rule, its severity in each profile,
       why it exists and how to fix it. See `rules`_ and
       :doc:`check-rules`.
   * - ``table dump``
     - Prints the rows of an encoded "Enter data" table, the one way to see
       what a mapping table holds. See `table dump`_.
   * - ``report blank``
     - Creates a blank report bound to a model that already exists. See
       `report blank`_.
   * - ``clone-report``
     - Copies a finished report folder under a new name and a new identity.
       See `clone-report`_.
   * - ``f-filters``
     - Fills the ``f-<Category>`` gate measures and the ``f-All`` gate lines
       of a model built on the selector pattern. See `f-filters`_.
   * - ``i-filters``
     - Regenerates the ``i-Applied Filters`` caption measure. See
       `i-filters`_.

The last five read or edit an existing project and are the commands you
reach for when a client model, a report or a helper measure needs work.
The tool's own repository documents them in depth, in
``docs/model-commands.md``:
https://github.com/C-Kapsalis/pbip-model-forge/blob/v1.2.0/docs/model-commands.md
This page gives you the command list, what each one does, the flags that
matter and what the tool prints. The transcripts below are the tool's own
output, with the absolute paths shortened to the folder's name.

The three spec commands
-------------------------

check
~~~~~~

::

    pbip-forge check SPEC [--profile {mockup,handoff}] [--format {text,json}]

``SPEC`` is a JSON file holding the spec dict, or a Python file that
defines a module-level ``SPEC`` dict, like the example scripts in the
tool's repository (``scripts/build_vet_clinic.py``).

.. list-table::
   :widths: 22 14 64
   :header-rows: 1

   * - Option
     - Default
     - Meaning
   * - ``--profile``
     - ``mockup``
     - The strictness profile: ``mockup`` or ``handoff``. It decides which
       rules are P1 and which are advisory P2. See :doc:`check-rules`.
   * - ``--format``
     - ``text``
     - ``text`` prints one line per violation and a summary; ``json``
       prints the same findings as a JSON object.

The text report prints one line per violation, the fix indented below it,
and a summary line:

::

    P1 FM015 'Clinic Measures'[Avg Billed per Patient]: DIVIDE denominator counts dimension table 'Patients', so filters from other dimensions do not reach it
        fix: Count from the fact instead: DISTINCTCOUNT(Encounters[patient_id]).
    pbip-forge check (handoff): 0 P0, 1 P1, 0 P2

The JSON report carries the profile and every violation:

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

The command exits 1 when any P0 or P1 violation is found, and 0
otherwise. A spec that cannot be loaded ends the run with exit 1: a file
that is neither ``.json`` nor ``.py`` prints ``<path>: expected a .json or
.py spec file``, and a ``.py`` file with no module-level ``SPEC`` prints
``<path> does not define a module-level SPEC dict``. A usage error (no
spec, an unknown option) exits 2.

build
~~~~~

::

    pbip-forge build SPEC OUT_DIR [--check] [--profile {mockup,handoff}]
                            [--no-handoff]

Writes the model into ``OUT_DIR``. The spec is read exactly as ``check``
reads it, and the two argument positions are fixed: the spec first, the
output folder second.

.. list-table::
   :widths: 22 14 64
   :header-rows: 1

   * - Option
     - Default
     - Meaning
   * - ``--check``
     - off
     - Refuse to build when the check finds a P0 or P1 violation. The
       check report is printed and nothing is written.
   * - ``--profile``
     - ``mockup``
     - The profile the check runs under.
   * - ``--no-handoff``
     - off
     - Skip ``handoff/manifest.json`` and ``handoff/SPEC.md``.

A successful build prints one line:

::

    Built PBIP: /path/to/out/VetClinic/VetClinic.pbip

and leaves this tree behind:

::

    out/VetClinic/
        VetClinic.pbip
        VetClinic.SemanticModel/
            .platform
            definition.pbism
            definition/
                model.tmdl
                database.tmdl
                relationships.tmdl
                tables/*.tmdl
        VetClinic.Report/
            .platform
            definition.pbir
        handoff/
            SPEC.md
            manifest.json

The report folder the builder writes is a two-file stub: ``.platform`` and
``definition.pbir``, the latter pointing at
``../VetClinic.SemanticModel``. It has no pages and no visuals, which is
the point — the deliverable is the model. ``pbip-forge report blank``
writes a complete report definition instead; see `report blank`_.

With ``--check``, a spec that fails the gate writes nothing at all and
exits 1:

::

    P1 FM015 'Clinic Measures'[Avg Billed per Patient]: DIVIDE denominator counts dimension table 'Patients', so filters from other dimensions do not reach it
        fix: Count from the fact instead: DISTINCTCOUNT(Encounters[patient_id]).
    pbip-forge check (handoff): 0 P0, 1 P1, 0 P2

From Python, the same build is ``build_pbip(spec, out_dir, check=True,
profile="handoff")``; it raises ``SpecCheckError`` without writing
anything when the check fails.

rules
~~~~~

::

    pbip-forge rules

Prints the 22 rules of the catalog, each with its name, its severity in
both profiles, why it exists and how to fix it. It takes no options and
always exits 0:

::

    FM001  relationship-endpoints     mockup P0  handoff P0
          A relationship whose table or column does not exist cannot be built.
          fix: Correct the table or column name in the relationship, or add the column.

    FM002  row-width                  mockup P0  handoff P0
          Every sample row must have one value per declared column, or the entered-data partition loads shifted or missing values.
          fix: Add or remove values so each row matches the column list.

The full catalog, with every rule's prose, is in :doc:`check-rules`.

The five project commands
---------------------------

The commands below take a PBIP project that already exists. ``f-filters``
and ``i-filters`` edit files: they take a timestamped backup of the file
they change unless ``--no-backups`` is given, keep the file's line endings
and its UTF-8 BOM, and change only the lines they insert or replace. Work
in a git repository, or on a copy, so every change can be undone.

table dump
~~~~~~~~~~

::

    pbip-forge table dump FILE.tmdl [--max-rows N]
    pbip-forge table dump --b64 STRING [--max-rows N]

Power BI stores an "Enter data" table as ``Binary.FromText("<base64>")``:
a raw DEFLATE stream of a JSON array of rows. ``table dump`` decodes the
first such payload in the file and prints it. The file is never written.

.. list-table::
   :widths: 22 14 64
   :header-rows: 1

   * - Option
     - Default
     - Meaning
   * - ``--max-rows``
     - 20
     - How many data rows to print.
   * - ``--b64``
     - none
     - Read a bare base64 payload instead of a file. The file argument and
       this option are alternatives; one of them is required.

The dump prints the column names (row 0 of the payload), the number of
data rows, and the first ``N`` rows numbered from 1:

::

    Columns: ['1', 'Max', 'Dog', 'Labrador', 'John Carter']

    Total rows: 7

    Data (first 20 rows):
    --------------------------------------------------------------------------------
      1. ['2', 'Bella', 'Cat', 'Siamese', 'Maria Lopez']
      2. ['3', 'Rocky', 'Dog', 'Bulldog', 'Tom Reed']
      3. ['4', 'Whiskers', 'Cat', 'Persian', 'Anna Kim']
      4. ['5', 'Charlie', 'Rabbit', 'Dutch', 'Sam Patel']
      5. ['6', 'Luna', 'Dog', 'Poodle', 'Emily Chen']
      6. ['7', 'Milo', 'Cat', 'Tabby', 'David Brown']
      7. ['8', 'Coco', 'Bird', 'Cockatiel', 'Rachel Green']

With ``--max-rows 2`` the last line becomes ``... and 5 more rows``. A
file with no payload prints ``No binary data found in <file>`` on standard
error and exits 1; a payload that does not decode prints ``Error
decrypting: <the error>`` and exits 1.

report blank
~~~~~~~~~~~~

::

    pbip-forge report blank --client DIR --model NAME [NAME ...]
                            [--name REPORT] [--theme NAME] [--theme-file PATH]
                            [--full] [--dry-run]

Creates ``<DIR>/<REPORT>.Report`` bound to ``<DIR>/<NAME>.SemanticModel``:
the report a Fabric Git sync or a deployment pipeline needs next to a
model that already exists.

.. list-table::
   :widths: 22 14 64
   :header-rows: 1

   * - Option
     - Default
     - Meaning
   * - ``--client``
     - required
     - The folder holding ``<name>.SemanticModel``.
   * - ``--model``
     - required
     - The model name or names. One report is created per name.
   * - ``--name``
     - the model name
     - The report name. Only valid with exactly one ``--model``.
   * - ``--theme``
     - ``CY25SU12``
     - The base theme name written into ``report.json``.
   * - ``--theme-file``
     - none
     - A theme JSON to copy into the report's
       ``StaticResources/SharedResources/BaseThemes`` folder.
   * - ``--full``
     - accepted
     - Accepted and has no further effect: the command always writes the
       complete definition.
   * - ``--dry-run``
     - off
     - List what would be created and write nothing.

The command writes the complete PBIR definition: ``.platform``,
``definition.pbir``, ``definition/report.json``,
``definition/version.json``, one page (``Page 1``, 1280 x 720) listed in
``definition/pages/pages.json``, the ``bookmarks`` and ``StaticResources``
folders, and ``.pbi/localSettings.json``. Every file is written with
two-space indentation and no trailing newline. The ``logicalId`` and the
page id are random, so two runs differ only there.

The console output is a banner, one block per model, and a footer:

::

    ================================================================================
    CREATE BLANK REPORTS
    ================================================================================

    Creating blank report: VetClinic
      Location: /path/to/client/VetClinic.Report
      References: VetClinic.SemanticModel

    ✅ Created blank report: VetClinic
       Location: /path/to/client/VetClinic.Report

    ================================================================================
    ✅ ALL REPORTS CREATED SUCCESSFULLY
    ================================================================================

A report folder that already exists is never overwritten; the run prints
``ℹ️  Report already exists: <name>`` and skips it. A model folder that
does not exist is skipped the same way, with
``⚠️  Semantic model not found: <path>``. A missing ``--client`` folder
prints ``❌ Client directory not found: <path>`` and exits 1. ``--name``
with more than one ``--model`` is a usage error and exits 2.

clone-report
~~~~~~~~~~~~

::

    pbip-forge clone-report TEMPLATE [NAME] [TARGET_DIR] [SEMANTIC_MODEL]
                            [--model MODEL] [--force] [--logical-id UUID]
                            [--strict] [--expect-visuals N]
                            [--expect-bookmark NAME]

Copies a finished report folder — hidden files included — and gives the
copy a new identity. ``TARGET_DIR`` is the report folder itself, used
exactly as given: no ``.Report`` is added to it.

.. list-table::
   :widths: 22 14 64
   :header-rows: 1

   * - Argument
     - Default
     - Meaning
   * - ``TEMPLATE``
     - required
     - The report folder to copy.
   * - ``NAME``
     - ``MyCompany-Analytics``
     - The deployment name. It becomes the copy's display name.
   * - ``TARGET_DIR``
     - ``<cwd>/<NAME>``
     - Where the copy is written.
   * - ``SEMANTIC_MODEL``
     - ``<NAME>.SemanticModel``
     - The model the copy points at. A free string: no existence check.

.. list-table::
   :widths: 22 14 64
   :header-rows: 1

   * - Option
     - Default
     - Meaning
   * - ``--model MODEL``
     - none
     - The same as the ``SEMANTIC_MODEL`` argument. The positional wins
       when both are given.
   * - ``--force``
     - off
     - Replace an existing target without asking.
   * - ``--logical-id UUID``
     - random
     - The ``logicalId`` to use instead of a generated one.
   * - ``--strict``
     - off
     - Accepted, and it changes nothing: the four required files are
       checked on every run, and a missing one ends the run with exit 1
       with or without the flag.
   * - ``--expect-visuals N``
     - none
     - Count the copy's visual files and warn when the count differs. The
       run still succeeds.
   * - ``--expect-bookmark NAME``
     - none
     - Check that this bookmark exists in the copy. A missing one fails
       the run. Repeatable.

The copy keeps a ``.backup`` of every file it edits: ``definition.pbir``
and ``.platform``. ``definition.pbir`` is pointed at
``../<SEMANTIC_MODEL>``; ``.platform`` gets a new random ``logicalId`` and
the new display name. Only the value's bytes change: the formatting and
the key order of both files stay as they were.

When the target exists, the command asks
``Continue and overwrite? (y/N): `` and only ``y`` or ``Y`` continues.
``--force`` answers the prompt with yes.

The four required files — ``definition.pbir``, ``.platform``,
``definition/report.json`` and ``definition/version.json`` — are checked
in that order after the copy. A missing one ends the run with exit 1, and
the copy stays where it is.

Every log line carries a colored tag, ``[INFO]``, ``[SUCCESS]``,
``[WARNING]`` or ``[ERROR]``:

::

    [INFO] Configuration:
    [INFO]   Deployment Name: VetClinic-Analytics
    [INFO]   Target Directory: /path/to/clone/VetClinic-Analytics.Report
    [INFO]   Semantic Model: VetClinic.SemanticModel
    [INFO]   Template Source: /path/to/client/VetClinic.Report

    [INFO] Validating prerequisites...
    [SUCCESS] Prerequisites validated
    [INFO] Deploying universal M Template...
    [INFO] Updating semantic model reference to: VetClinic.SemanticModel
    [SUCCESS] Updated semantic model reference
    [INFO] Generating unique logical ID...
    [SUCCESS] Generated unique logical ID: e3a25895-4a42-4c37-9c2a-58ffd5c3f030
    [SUCCESS] Universal M Template deployed successfully!
    [INFO] Validating deployment...
    [SUCCESS] Required file exists: definition.pbir
    [SUCCESS] Required file exists: .platform
    [SUCCESS] Required file exists: definition/report.json
    [SUCCESS] Required file exists: definition/version.json
    [SUCCESS] Deployment validation completed!
    [SUCCESS] Universal M Template deployment completed successfully!

``--expect-visuals`` warns when the count differs and lets the run
succeed:

::

    [INFO] Validating deployment...
    [WARNING] Visual count: 0 (expected: 5)
    [WARNING] x/y/t Protocol validation incomplete

``--expect-bookmark`` fails the run when the bookmark is missing:

::

    ❌ definition/bookmarks/Missing.bookmark.json: MISSING
    ❌ VALIDATION FAILED - Fix critical issues before pushing to Power BI Online

Two copies with the same ``logicalId`` collide on Fabric Git sync, which
is why the clone always gets its own.

.. list-table::
   :widths: 14 86
   :header-rows: 1

   * - Exit code
     - When
   * - 0
     - The copy was made, or the operator declined the overwrite prompt.
   * - 1
     - The template is missing, a required file is missing, or the
       deployment check failed.
   * - 2
     - The target lies inside the template, or ``--logical-id`` is not a
       UUID.

f-filters
~~~~~~~~~

::

    pbip-forge f-filters (--model M.SemanticModel | --fleet DIR) [--dry-run]
                            [--no-backups] [-v]

For models built on the selector pattern: one hidden ``f-<Category>``
measure per analysis category in the ``f-Filters`` table, and an ``f-All``
measure whose two chains of gate lines (``VAR temp_basic`` joined with
``&&``, ``VAR temp_alt`` joined with ``||``) offer the canvas only the
categories that hold data.

.. list-table::
   :widths: 22 14 64
   :header-rows: 1

   * - Option
     - Default
     - Meaning
   * - ``--model``
     - none
     - A ``<X>.SemanticModel`` folder. Repeatable. Exactly one of
       ``--model`` and ``--fleet`` is required.
   * - ``--fleet``
     - none
     - A folder whose ``*.SemanticModel`` children are all processed.
   * - ``--dry-run``
     - off
     - Show the plan, write nothing.
   * - ``--no-backups``
     - off
     - Do not keep a timestamped backup.
   * - ``-v``
     - off
     - Print the steps: the uncovered facts, the created DAX, the gate
       passes and the file written.

The command finds two kinds of gap and repairs both by appending only: a
fact table that a user-facing measure reads but that no ``f-`` measure
covers, and a category registered in the ``z-Map`` table that has no gate
line in ``f-All``. Existing lines are never rewritten.

The run prints a header, one block per model, and a summary line:

::

    f-filter update — 1 target(s); mode: APPLY

    ── /path/to/Roastery.SemanticModel  (/path/to/Roastery.SemanticModel)

       ⚠️  EXTENDED f-Spend by appending `+ int(not ISEMPTY('Web Sales'))`
         If the existing measure is AGGREGATOR, SWITCH-ROUTER, or DISPLAY-STRING (per tmdl-preflight A005 f-measure-pattern), this append likely BROKE its semantics or DAX syntax. MANUAL VERIFY NOW before committing.
       created=0 extended=1 skipped=0 f-All=-  -> OK
         EXTEND  f-Spend  (+ Web Sales)

The summary line counts what happened: ``created``, ``extended``,
``skipped``, and ``f-All`` (the gate lines added, with ``skip`` for the
categories that were already gated). Each created, extended or skipped
item gets its own line below it. A fact table with no category is skipped
with the reason spelled out.

A model with no ``f-Filters.tmdl`` fails, and the run exits 1:

::

       created=0 extended=0 skipped=0 f-All=-  -> FAIL: f-Filters.tmdl not found: /path/to/Roastery.SemanticModel/definition/tables/f-Filters.tmdl

The backup is ``f-Filters.tmdl.backup_gapfill_<YYYYmmdd_HHMMSS>``, a byte
copy of the file before the edit. Standard error always ends with a
review block that names the three other shapes an ``f-`` measure can
take, and the classifier to run afterwards:
``tmdl-preflight check <model> --select A005``.

i-filters
~~~~~~~~~

::

    pbip-forge i-filters (--model M.SemanticModel | --fleet DIR) [--dry-run]
                            [--no-backups] [-v] [--skip-validation]

Regenerates the ``i-Applied Filters`` caption measure in ``Helper
Measures`` from the ``y-Selector*`` and ``y-Plot Specific*`` tables: the
caption a report shows above a visual, naming the filters currently
applied.

.. list-table::
   :widths: 22 14 64
   :header-rows: 1

   * - Option
     - Default
     - Meaning
   * - ``--model``
     - none
     - A ``<X>.SemanticModel`` folder. Repeatable. Exactly one of
       ``--model`` and ``--fleet`` is required.
   * - ``--fleet``
     - none
     - A folder whose ``*.SemanticModel`` children are all processed.
   * - ``--dry-run``
     - off
     - Show the plan, write nothing.
   * - ``--no-backups``
     - off
     - Do not keep a timestamped backup.
   * - ``-v``
     - off
     - Print the four steps: discovery, generation, validation and
       update.
   * - ``--skip-validation``
     - off
     - Write even when the generated measure fails validation.

Every ``NAMEOF('T'[c])`` in the selector tables is a candidate, ordered by
the number that follows it and then by table and column. The column
``z``, columns containing ``blank`` and ``__Calendar`` tables are dropped,
hidden columns are dropped, and a column the model does not declare is
missing.

Before writing, the measure is validated: every dimension and every
missing column must have its ``ISFILTERED`` test, and ``ISFILTERED`` and
``CONCATENATEX`` must be present. A model without a ``y-`` table, or with
a ``y-`` row on a column that does not exist, fails and nothing is
written:

::

    ❌ Validation failed. Run with --verbose for details.
       dims=2 gen=1 updated=0 failed=0  -> FAIL: Validation failed - see details above

``--skip-validation`` leaves the missing columns out and writes anyway.
A missing ``Helper Measures.tmdl`` fails the model in the same way, with
``Helper Measures file not found: <path>``.

The run prints a header, one block per model, and a summary line:

::

    i-filter update — 1 target(s); mode: APPLY

    ── /path/to/Shops.SemanticModel  (/path/to/Shops.SemanticModel)
       dims=2 gen=1 updated=1 failed=0  -> OK

The summary counts the dimensions found, the measures generated, the
files updated and the models that failed. A dry run prints a preview
first: the line and character counts of the old and the new text, and
the first and last lines of each.

Only the measure's expression is replaced — fenced, bare or inline —
and everything else in the file stays byte for byte. An unchanged file is
neither rewritten nor backed up. The backup is
``Helper Measures.tmdl.backup_<YYYYmmdd_HHMMSS>``.

The two gates
--------------

A model is done when it passes both gates, in this order: ``pbip-forge
check`` on the spec, then ``tmdl-preflight`` on the files. The first is
part of this project; the second is a separate tool, installed once
alongside it.

::

    pbip-forge check scripts/build_vet_clinic.py --profile handoff
    pbip-forge check (handoff): 0 P0, 0 P1, 0 P2

    pbip-forge build scripts/build_vet_clinic.py out/VetClinic --check --profile handoff
    Built PBIP: /path/to/out/VetClinic/VetClinic.pbip

The semantic model the builder writes is clean. The report folder it
writes next to the model is a two-file stub, and ``tmdl-preflight``
v1.2.0 reads two of its report rules as errors on that stub: ``B007``
(missing ``report.json``) and ``B005`` (missing
``definition/pages/pages.json``). The run over the whole project
therefore reports those two errors and exits 1, while the model gate — the
same run with the two report rules ignored — reports none:

::

    tmdl-preflight check out/VetClinic
    VetClinic.Report  B007 error: VetClinic.Report: Missing report.json (neither at root nor in definition/)
    VetClinic.Report/definition/pages/pages.json  B005 error: definition/pages/pages.json is missing; the report has no pages metadata. Save the report again from Power BI Desktop (PBIR format). [VetClinic.Report]
    tmdl-preflight: 2 error(s), 0 warning(s), 1 info(s)

    tmdl-preflight check out/VetClinic --ignore B005,B007
    tmdl-preflight: 0 error(s), 0 warning(s), 1 info(s)

Both lines are worth reporting: the first names the two findings on the
report stub, the second is the model itself. ``pbip-forge report blank``
writes a complete report definition, the kind tmdl-preflight expects, for
a model that already exists — see `report blank`_.

Exit codes
-----------

.. list-table::
   :widths: 14 86
   :header-rows: 1

   * - Exit code
     - When
   * - 0
     - The run did what it was asked to do.
   * - 1
     - A gate failed, a file was missing, a target could not be written,
       or a check found a P0 or P1 violation.
   * - 2
     - A usage error: an unknown option, a missing argument, a
       ``--name`` with more than one ``--model``, a target inside the
       clone template, or a ``--logical-id`` that is not a UUID.
