Commands for existing projects
================================

``check``, ``build`` and ``rules`` work on a spec (:doc:`cli`). The five
commands on this page work on a PBIP project that already exists: they
read an encoded table, create or copy a report, or repair two helper
measures of the selector pattern. This page mirrors the tool's own
``docs/model-commands.md`` at v1.2.1:
https://github.com/C-Kapsalis/pbip-model-forge/blob/v1.2.1/docs/model-commands.md

``f-filters`` and ``i-filters`` edit files. Both take ``--dry-run``, and
both keep a timestamped backup of the file they change unless
``--no-backups`` is given. The backup is a byte copy of the file before
the edit. The edit keeps the file's line endings (CRLF stays CRLF, LF
stays LF) and a UTF-8 byte order mark, and changes only the lines it
inserts or replaces. Run them with ``--dry-run`` first, in a git
repository or on a copy, so every change can be undone.

The examples
--------------

The transcripts below were run from the root of the tool's repository,
cloned at v1.2.1 (:doc:`../how-to/install`), after building the
veterinary clinic example and copying two of the repository's test
fixtures into ``out``. The fixtures are small, invented models built on
the selector pattern: ``Roastery`` for ``f-filters`` and ``Shops`` for
``i-filters``. ``out`` is ignored by the repository's ``.gitignore``.

In PowerShell:

::

    pbip-forge build scripts/build_vet_clinic.py out/VetClinic --check --profile handoff
    Copy-Item -Recurse tests/fixtures/f_filters/Roastery.SemanticModel out/
    Copy-Item -Recurse tests/fixtures/i_filters/Shops.SemanticModel out/

In bash:

::

    pbip-forge build scripts/build_vet_clinic.py out/VetClinic --check --profile handoff
    cp -r tests/fixtures/f_filters/Roastery.SemanticModel out/
    cp -r tests/fixtures/i_filters/Shops.SemanticModel out/

Where a transcript shows an absolute path, it is shortened to
``/path/to/pbip-model-forge``.

table dump
------------

::

    pbip-forge table dump FILE.tmdl [--max-rows N]
    pbip-forge table dump --b64 STRING [--max-rows N]

Power BI stores an "Enter data" table as ``Binary.FromText("<base64>")``:
a raw DEFLATE stream of a JSON array of rows. ``table dump`` decodes the
first such payload in the file and prints it. It never writes the file.
It is the one way to see what an encoded mapping table holds.

.. list-table::
   :widths: 22 14 64
   :header-rows: 1

   * - Option
     - Default
     - Meaning
   * - ``FILE.tmdl``
     - none
     - The table file to read. Either the file or ``--b64`` is required.
   * - ``--b64``
     - none
     - A bare base64 payload to decode instead of a file.
   * - ``--max-rows``
     - 20
     - How many data rows to print.

::

    pbip-forge table dump out/VetClinic/VetClinic.SemanticModel/definition/tables/Patients.tmdl --max-rows 3

::

    Columns: ['1', 'Max', 'Dog', 'Labrador', 'John Carter']

    Total rows: 7

    Data (first 3 rows):
    --------------------------------------------------------------------------------
      1. ['2', 'Bella', 'Cat', 'Siamese', 'Maria Lopez']
      2. ['3', 'Rocky', 'Dog', 'Bulldog', 'Tom Reed']
      3. ['4', 'Whiskers', 'Cat', 'Persian', 'Anna Kim']
    ... and 4 more rows

The dump always prints row 0 of the payload as ``Columns`` and counts the
rest as data. A table written by ``build`` stores no header row, so for
these tables ``Columns`` is the first patient, and the table really holds
eight rows, not seven. A mapping table whose first row holds the column
names reads naturally.

An empty payload prints ``No data`` and exits 0. The errors, each with
exit 1:

.. list-table::
   :widths: 40 60
   :header-rows: 1

   * - Case
     - What is printed
   * - The file has no payload
     - ``No binary data found in <file>``, on standard error.
   * - The file does not exist
     - ``Error decrypting: the file could not be read: <the error>``.
   * - The file's payload does not decode
     - ``Error decrypting: <the error>``.
   * - The ``--b64`` payload does not decode
     - ``error: encoded payload does not decode: <the error>``, on
       standard error.
   * - The payload is not a JSON array of rows
     - ``error: encoded payload is not a JSON array of rows``, on
       standard error.

Neither the file nor ``--b64`` is a usage error and exits 2.

report blank
--------------

::

    pbip-forge report blank --client DIR --model NAME [NAME ...] [--name REPORT]
                            [--theme NAME] [--theme-file PATH] [--full] [--dry-run]

Creates ``<DIR>/<REPORT>.Report`` bound to ``<DIR>/<NAME>.SemanticModel``:
the report a Fabric Git sync or a deployment pipeline needs next to a
model that has none.

.. list-table::
   :widths: 22 18 60
   :header-rows: 1

   * - Option
     - Default
     - Meaning
   * - ``--client``
     - required
     - The folder that holds ``<NAME>.SemanticModel``.
   * - ``--model``
     - required
     - One or more model names. One report is created per name.
   * - ``--name``
     - the model name
     - The report name. Only valid with exactly one ``--model``.
   * - ``--theme``
     - ``CY25SU12``
     - The base theme named in ``report.json``.
   * - ``--theme-file``
     - none
     - A theme JSON to copy into
       ``StaticResources/SharedResources/BaseThemes/<theme>.json``.
   * - ``--full``
     - off
     - Accepted, with no further effect: the command always writes the
       complete definition.
   * - ``--dry-run``
     - off
     - List what would be created and write nothing.

The report it writes: ``.platform``, ``definition.pbir`` (pointing at
``../<NAME>.SemanticModel``), ``definition/report.json``,
``definition/version.json``, one page (``Page 1``, 1280 x 720) listed in
``definition/pages/pages.json`` with the pages metadata schema 1.1.0, and
the ``definition/bookmarks`` and ``StaticResources`` folders. It does not
write ``.pbi/localSettings.json``, which Power BI Desktop writes itself
and which belongs in ``.gitignore``. Every file uses two-space indentation
and has no trailing newline. The ``logicalId`` and the page id are
random, so two runs differ only there; ``build`` uses fixed ones instead.

The theme JSON is copied from ``--theme-file``, or else from another
report in the same ``--client`` folder that has the theme. When neither
exists, ``report.json`` still names the theme and no theme file is
written.

To try it, copy the clinic's model into a folder of its own. In
PowerShell:

::

    mkdir out/client
    Copy-Item -Recurse out/VetClinic/VetClinic.SemanticModel out/client/

In bash:

::

    mkdir out/client
    cp -r out/VetClinic/VetClinic.SemanticModel out/client/

A dry run lists the files and folders it would create:

::

    pbip-forge report blank --client out/client --model VetClinic --dry-run

::

    ================================================================================
    CREATE BLANK REPORTS
    ================================================================================

    Creating blank report: VetClinic
      Location: out/client/VetClinic.Report
      References: VetClinic.SemanticModel

      [DRY RUN] Would create:
        - out/client/VetClinic.Report/.platform
        - out/client/VetClinic.Report/definition.pbir
        - out/client/VetClinic.Report/definition/report.json
        - out/client/VetClinic.Report/definition/version.json
        - out/client/VetClinic.Report/definition/pages/
        - out/client/VetClinic.Report/definition/bookmarks/
        - out/client/VetClinic.Report/StaticResources/

    ================================================================================
    ✅ ALL REPORTS CREATED SUCCESSFULLY
    ================================================================================

The dry run ends with the same footer as a real run, though it creates
nothing.

The closing line tells you how the run went:

.. list-table::
   :widths: 40 50 10
   :header-rows: 1

   * - Closing line
     - When
     - Exit
   * - ``✅ ALL REPORTS CREATED SUCCESSFULLY``
     - Every model got its report or already had one.
     - 0
   * - ``⚠️  SOME REPORTS FAILED``
     - At least one report was created and at least one model failed.
     - 1
   * - ``⚠️  NO REPORTS CREATED``
     - No report was created and at least one model failed.
     - 1

A model fails when its
``.SemanticModel`` folder does not exist or when writing its report
raises an error; the other models are still processed. A report folder
that already exists is never overwritten: it is skipped, and a skip is
not a failure.

Two models, one of which does not exist:

::

    pbip-forge report blank --client out/client --model VetClinic Hospital

::

    ================================================================================
    CREATE BLANK REPORTS
    ================================================================================

    Creating blank report: VetClinic
      Location: out/client/VetClinic.Report
      References: VetClinic.SemanticModel

    ✅ Created blank report: VetClinic
       Location: out/client/VetClinic.Report

    ⚠️  Semantic model not found: out/client/Hospital.SemanticModel
       Skipping report creation for Hospital

    ================================================================================
    ⚠️  SOME REPORTS FAILED
    ================================================================================

The same command for ``VetClinic`` alone now skips the existing report
and exits 0:

::

    pbip-forge report blank --client out/client --model VetClinic

::

    ================================================================================
    CREATE BLANK REPORTS
    ================================================================================

    ℹ️  Report already exists: VetClinic
       Skipping: out/client/VetClinic.Report

    ================================================================================
    ✅ ALL REPORTS CREATED SUCCESSFULLY
    ================================================================================

For ``Hospital`` alone, nothing is created and the run exits 1:

::

    pbip-forge report blank --client out/client --model Hospital

::

    ================================================================================
    CREATE BLANK REPORTS
    ================================================================================

    ⚠️  Semantic model not found: out/client/Hospital.SemanticModel
       Skipping report creation for Hospital

    ================================================================================
    ⚠️  NO REPORTS CREATED
    ================================================================================

A ``--client`` folder that does not exist prints ``❌ Client directory not
found: <path>`` and exits 1. ``--name`` with more than one ``--model``
prints ``error: --name works with exactly one --model`` and exits 2.

The new report passes tmdl-preflight:

::

    tmdl-preflight check out/client/VetClinic.Report

::

    scanned: 0 models, 0 tables, 0 columns, 0 measures, 0 relationships, 0 DAX expressions, 1 report
    No .bookmark.json files found under out/client/VetClinic.Report
    tmdl-preflight: 0 error(s), 0 warning(s), 0 info(s)

clone-report
--------------

::

    pbip-forge clone-report TEMPLATE [NAME] [TARGET_DIR] [SEMANTIC_MODEL]
                            [--model MODEL] [--force] [--logical-id UUID] [--strict]
                            [--expect-visuals N] [--expect-bookmark NAME]

Copies a report folder, hidden files included, and gives the copy a new
identity. Two reports with the same ``logicalId`` collide on Fabric Git
sync, which is why the copy always gets its own.

.. list-table::
   :widths: 22 22 56
   :header-rows: 1

   * - Argument
     - Default
     - Meaning
   * - ``TEMPLATE``
     - required
     - The report folder to copy.
   * - ``NAME``
     - ``MyCompany-Analytics``
     - The new display name.
   * - ``TARGET_DIR``
     - ``<current folder>/<NAME>``
     - The report folder to create, used exactly as given: no ``.Report``
       is added. Missing parent folders are created.
   * - ``SEMANTIC_MODEL``
     - ``<NAME>.SemanticModel``
     - The model folder name the copy points at, as
       ``../<SEMANTIC_MODEL>``. It is not checked: the folder need not
       exist.

.. list-table::
   :widths: 22 14 64
   :header-rows: 1

   * - Option
     - Default
     - Meaning
   * - ``--model MODEL``
     - none
     - The same as ``SEMANTIC_MODEL``. The positional argument wins when
       both are given.
   * - ``--force``
     - off
     - Replace an existing target without asking.
   * - ``--logical-id UUID``
     - random
     - The ``logicalId`` to give the copy.
   * - ``--strict``
     - off
     - Accepted, but it changes nothing in 1.2.1. Its help text says it
       makes the run exit 1 when the copy fails the report integrity
       checks, but a copy whose ``pages.json`` lists a missing page, or
       whose model does not exist, still exits 0 with ``--strict``. Run ``tmdl-preflight check`` on the copy instead.
   * - ``--expect-visuals N``
     - none
     - Count the copy's visual files and warn when the count differs. The
       run still succeeds.
   * - ``--expect-bookmark NAME``
     - none
     - Check that ``definition/bookmarks/<NAME>.bookmark.json`` exists in
       the copy; a missing one fails the run. Repeatable.

The command edits two files in the copy and keeps the original of each
as ``<file>.backup`` next to it: ``definition.pbir`` gets the new model
path and ``.platform`` gets the new ``logicalId`` and display name. Only
those values change; the formatting and key order of both files stay as
they were. Then it checks that ``definition.pbir``, ``.platform``,
``definition/report.json`` and ``definition/version.json`` exist, in that
order. A missing one prints ``[ERROR] Missing required file: <file>`` and
ends the run with exit 1; the copy stays on disk.

When the target exists and ``--force`` is not given, the command prints
``Continue and overwrite? (y/N):`` and goes on only after ``y`` or ``Y``.
Any other answer prints ``[INFO] Deployment cancelled.`` and exits 0.

A second report for the clinic model, next to the first:

::

    pbip-forge clone-report out/VetClinic/VetClinic.Report VetClinic-Analytics out/VetClinic/VetClinic-Analytics.Report VetClinic.SemanticModel

::

    [INFO] Configuration:
    [INFO]   Deployment Name: VetClinic-Analytics
    [INFO]   Target Directory: out/VetClinic/VetClinic-Analytics.Report
    [INFO]   Semantic Model: VetClinic.SemanticModel
    [INFO]   Template Source: out/VetClinic/VetClinic.Report

    [INFO] Validating prerequisites...
    [SUCCESS] Prerequisites validated
    [INFO] Deploying universal M Template...
    [INFO] Updating semantic model reference to: VetClinic.SemanticModel
    [SUCCESS] Updated semantic model reference
    [INFO] Generating unique logical ID...
    [SUCCESS] Generated unique logical ID: 5e361c0d-0b03-4986-b724-d59bbefd00fe
    [SUCCESS] Universal M Template deployed successfully!
    [INFO] Validating deployment...
    [SUCCESS] Required file exists: definition.pbir
    [SUCCESS] Required file exists: .platform
    [SUCCESS] Required file exists: definition/report.json
    [SUCCESS] Required file exists: definition/version.json
    [SUCCESS] Deployment validation completed!
    [SUCCESS] Universal M Template deployment completed successfully!

The ``logicalId`` differs on every run. The tags ``[INFO]``,
``[SUCCESS]``, ``[WARNING]`` and ``[ERROR]`` are printed with terminal
color codes, also when the output is redirected to a file.

``--expect-visuals`` and ``--expect-bookmark`` also print a banner before
the configuration and a block of next steps after the checks. With
``--expect-visuals 5`` on a report with no visuals, the check prints:

::

    [INFO] Validating deployment...
    [WARNING] Visual count: 0 (expected: 5)
    [WARNING] x/y/t Protocol validation incomplete

and the run exits 0. With ``--expect-bookmark Missing``:

::

    ❌ definition/bookmarks/Missing.bookmark.json: MISSING
    ❌ VALIDATION FAILED - Fix critical issues before pushing to Power BI Online

and the run exits 1.

.. list-table::
   :widths: 14 86
   :header-rows: 1

   * - Exit
     - When
   * - 0
     - The copy was made, or the overwrite prompt was declined.
   * - 1
     - The template does not exist (``[ERROR] Universal template not
       found at: <path>``), a required file is missing, or an expected
       bookmark is missing.
   * - 2
     - The target lies inside the template, or ``--logical-id`` is not a
       UUID (``[ERROR] --logical-id is not a UUID: '<value>'``).

f-filters
-----------

::

    pbip-forge f-filters (--model M.SemanticModel | --fleet DIR) [--dry-run]
                         [--no-backups] [-v]

For models built on the selector pattern. The ``f-Filters`` table holds
one hidden ``f-<Category>`` measure per analysis category, and an
``f-All`` measure with two chains of gate lines: ``VAR temp_basic``,
joined with ``&&``, and ``VAR temp_alt``, joined with ``||``. The command
finds the gaps and fills them by appending only; it never rewrites an
existing line.

.. list-table::
   :widths: 22 14 64
   :header-rows: 1

   * - Option
     - Default
     - Meaning
   * - ``--model``
     - none
     - A ``<X>.SemanticModel`` folder. Repeatable. Give ``--model`` or
       ``--fleet``, not both.
   * - ``--fleet``
     - none
     - A folder whose ``*.SemanticModel`` children are all processed.
   * - ``--dry-run``
     - off
     - Show the plan and write nothing.
   * - ``--no-backups``
     - off
     - Do not keep a backup.
   * - ``-v``, ``--verbose``
     - off
     - Print every step.

What it repairs:

- **A fact table no f- measure covers.** A fact table that a user-facing
  measure reads, that no ``f-`` measure reads, and that no mapped
  ``x-Selector`` entry already gates, is added to the ``f-<Category>``
  measure of each of its categories. An existing measure is extended with
  ``+ int(not ISEMPTY('<Fact>'))``, with a warning; a missing one is
  created, hidden, in the display folder ``Visual-Level filters``. A fact
  with no category is skipped, with the reason printed.
- **A category with no gate line in f-All.** First for the categories it
  just created, then for every category registered in the ``z-Map
  Dimension-Measure Groupings`` table, the command appends one line to
  each chain of ``f-All``. A category counts as gated when an uncommented
  ``CONTAINSSTRING(all_cats, "...")`` line in ``temp_basic`` already
  names it.

The tool's ``docs/model-commands.md`` gives the exact rules for what
counts as a fact table, a user-facing measure and a category.

A dry run on the ``Roastery`` fixture, standard output only:

::

    pbip-forge f-filters --model out/Roastery.SemanticModel --dry-run

::

    f-filter update — 1 target(s); mode: DRY-RUN

    ── out/Roastery.SemanticModel  (/path/to/pbip-model-forge/out/Roastery.SemanticModel)

       ⚠️  EXTENDED f-Spend by appending `+ int(not ISEMPTY('Web Sales'))`
         If the existing measure is AGGREGATOR, SWITCH-ROUTER, or DISPLAY-STRING (per tmdl-preflight A005 f-measure-pattern), this append likely BROKE its semantics or DAX syntax. MANUAL VERIFY NOW before committing.
       created=0 extended=1 skipped=0 f-All=-  -> OK
         EXTEND  f-Spend  (+ Web Sales)

The summary line counts the measures created, the measures extended,
the facts skipped, and ``f-All``: the gate lines added for the
categories created in this run (``+<n>``, with ``/skip <n>`` for those
already gated), or ``-`` when no category was created. Gate lines added
for categories registered in ``z-Map`` are not in that count; ``-v``
shows them. Here the run also adds a gate line for ``Spend``, as the
verbose run shows:

::

    pbip-forge f-filters --model out/Roastery.SemanticModel -v

::

    f-filter update — 1 target(s); mode: APPLY

    ── out/Roastery.SemanticModel  (/path/to/pbip-model-forge/out/Roastery.SemanticModel)

    ── Loading model index from out/Roastery.SemanticModel
       Uncovered fact tables: 1

       ⚠️  EXTENDED f-Spend by appending `+ int(not ISEMPTY('Web Sales'))`
         If the existing measure is AGGREGATOR, SWITCH-ROUTER, or DISPLAY-STRING (per tmdl-preflight A005 f-measure-pattern), this append likely BROKE its semantics or DAX syntax. MANUAL VERIFY NOW before committing.

       f-All coverage: 2 z-Map categor(ies), 1 gate(s) added, 1 already gated, 0 unresolved
          + gate Spend -> [f-Spend]

       backup: f-Filters.tmdl.backup_gapfill_20261022_193337

       Wrote out/Roastery.SemanticModel/definition/tables/f-Filters.tmdl
       created=0 extended=1 skipped=0 f-All=-  -> OK
         EXTEND  f-Spend  (+ Web Sales)

The backup's name carries the time of the run. A second run finds
nothing to do:

::

    pbip-forge f-filters --model out/Roastery.SemanticModel

::

    f-filter update — 1 target(s); mode: APPLY

    ── out/Roastery.SemanticModel  (/path/to/pbip-model-forge/out/Roastery.SemanticModel)
       created=0 extended=0 skipped=0 f-All=-  -> OK

Standard error carries the extension warnings again and, on every run,
a review block that names the three other shapes an ``f-`` measure can
take and the command that classifies them:
``tmdl-preflight check <model> --select A005``. Run it after every
``f-filters`` run that extends a measure.

A model without ``f-Filters.tmdl`` fails, the other models still run,
and the command exits 1:

::

    pbip-forge f-filters --model out/VetClinic/VetClinic.SemanticModel

::

    f-filter update — 1 target(s); mode: APPLY

    ── out/VetClinic/VetClinic.SemanticModel  (/path/to/pbip-model-forge/out/VetClinic/VetClinic.SemanticModel)
       created=0 extended=0 skipped=0 f-All=-  -> FAIL: f-Filters.tmdl not found: out/VetClinic/VetClinic.SemanticModel/definition/tables/f-Filters.tmdl

A ``--fleet`` folder that does not exist, or holds no
``*.SemanticModel`` folder, prints ``0 target(s)`` and exits 0.

i-filters
-----------

::

    pbip-forge i-filters (--model M.SemanticModel | --fleet DIR) [--dry-run]
                         [--no-backups] [-v] [--skip-validation]

Regenerates the ``i-Applied Filters`` measure in the ``Helper Measures``
table: the caption a report shows above a visual, naming the filters
currently applied. The measure is built from the ``y-Selector*`` and
``y-Plot Specific*`` tables.

.. list-table::
   :widths: 22 14 64
   :header-rows: 1

   * - Option
     - Default
     - Meaning
   * - ``--model``
     - none
     - A ``<X>.SemanticModel`` folder. Repeatable. Give ``--model`` or
       ``--fleet``, not both.
   * - ``--fleet``
     - none
     - A folder whose ``*.SemanticModel`` children are all processed.
   * - ``--dry-run``
     - off
     - Print a preview and write nothing.
   * - ``--no-backups``
     - off
     - Do not keep a backup.
   * - ``-v``, ``--verbose``
     - off
     - Print the four steps: discovery, generation, validation and
       update.
   * - ``--skip-validation``
     - off
     - Write even when the generated measure fails validation, leaving
       the missing columns out.

Every ``NAMEOF('Table'[Column])`` in the selector tables is a candidate,
ordered by the number that follows it, then by table and column. The
column ``z``, columns whose name contains ``blank``, ``__Calendar``
columns and hidden columns are dropped. A column the model does not
declare is missing. For each remaining column the caption shows
``Column = a, b, c`` for the selected values (at most 10, a blank shown
as ``(Blank)``), or ``Column is not x`` when the column has more than 5
values and 1 to 3 are excluded.

Before writing, the command validates the measure: every column must
have its ``ISFILTERED`` test, and ``ISFILTERED`` and ``CONCATENATEX`` must
both appear. A model with no selector table, or with a selector row on a
column that does not exist, fails validation and nothing is written.

Only the measure's expression is replaced; everything else in the file
stays byte for byte. An unchanged file is neither rewritten nor backed
up. The backup is ``Helper Measures.tmdl.backup_<YYYYmmdd_HHMMSS>``.

On the ``Shops`` fixture:

::

    pbip-forge i-filters --model out/Shops.SemanticModel

::

    i-filter update — 1 target(s); mode: APPLY

    ── out/Shops.SemanticModel  (/path/to/pbip-model-forge/out/Shops.SemanticModel)
       dims=2 gen=1 updated=1 failed=0  -> OK

``dims`` is the number of columns in the caption, ``gen`` the measures
generated and ``updated`` the files written. Run it again and
``updated`` is 0, because the file already holds that measure.

A model without selector tables fails validation and exits 1:

::

    pbip-forge i-filters --model out/VetClinic/VetClinic.SemanticModel

::

    i-filter update — 1 target(s); mode: APPLY

    ── out/VetClinic/VetClinic.SemanticModel  (/path/to/pbip-model-forge/out/VetClinic/VetClinic.SemanticModel)

    ❌ Validation failed. Run with --verbose for details.
       dims=0 gen=1 updated=0 failed=0  -> FAIL: Validation failed - see details above

``failed`` stays 0 here although the model failed; read the status at
the end of the line, and the exit code. With ``-v``, the validation step
prints the coverage it found, for example ``❌ FAIL: i-Applied Filters
(66.7% coverage)``, but not the name of the missing column. A missing
``Helper Measures.tmdl`` fails with ``Helper Measures file not found:
<path>``.

A dry run prints a preview of the old and the new measure: line and
character counts and the first and last five lines of each. The "old"
side runs from the measure to the end of the file, so its counts include
the lines that follow the measure.

As with ``f-filters``, a ``--fleet`` folder with no ``*.SemanticModel``
folder prints ``0 target(s)`` and exits 0.
