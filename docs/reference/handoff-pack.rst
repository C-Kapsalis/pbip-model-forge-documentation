The hand-off pack
===================

Every build writes a ``handoff`` folder next to the ``.pbip`` file. It holds
what a data engineer needs to build the model for real, generated from the
same spec as the model itself:

::

    out/Hospital/
        Hospital.pbip
        Hospital.SemanticModel/
        Hospital.Report/
        handoff/
            SPEC.md
            manifest.json

Both files are deterministic: tables, relationships and measures are sorted
and nothing carries a timestamp, so rebuilding an unchanged model leaves
them byte-identical and a change shows up as a clean diff. Pass
``handoff=False`` to ``build_pbip`` (or ``--no-handoff`` to
``pbip-forge build``) to skip them.

The build command
------------------

The hand-off pack is written by ``pbip-forge build``, which takes the spec
first and the output folder second:

::

    pbip-forge build scripts/build_vet_clinic.py out/VetClinic --check --profile handoff
    Built PBIP: /path/to/out/VetClinic/VetClinic.pbip

``--check`` judges the spec before anything is written, and a spec with a
P0 or P1 violation writes nothing at all. ``--profile`` chooses the
strictness the check runs under: ``mockup`` by default, ``handoff`` for a
model you are about to hand over. ``--no-handoff`` skips the two files
this page describes. The whole command line is in :doc:`cli`.

SPEC.md
---------

The human-readable spec. The skills show it to you instead of drawing a
diagram by hand, so the diagram always matches the model. It contains:

- A Mermaid ``erDiagram`` of every table and relationship. Solid lines are
  active relationships, dotted lines inactive ones; key columns are marked
  ``PK`` and foreign keys ``FK``.
- A table list with each table's role (fact, dimension, calendar or
  measures) and description.
- A data dictionary per table: every column with its type, key, visibility,
  format and description.
- The relationships, with active and filter-direction flags.
- The measures, with their display folder, format, description and DAX.

The top of the hospital example's diagram:

::

    erDiagram
        __Calendar ||--o{ Encounters : "admission_date"
        Departments ||--o{ Encounters : "department_id"
        Diagnoses ||--o{ Encounters : "diagnosis_id"
        __Calendar ||..o{ Encounters : "discharge_date"
        Patients ||--o{ Encounters : "patient_id"
        Physicians ||--o{ Encounters : "physician_id"
        InsuranceCompanies ||--o{ Patients : "insurance_company_id"

manifest.json
---------------

The same model, machine-readable, for a data-engineering tool or a Claude
Code skill to build from. Top-level keys:

.. list-table::
   :widths: 20 80
   :header-rows: 1

   * - Key
     - Contents
   * - ``model``
     - The model name.
   * - ``generator``
     - The pbip-model-forge version that wrote the file.
   * - ``tables``
     - One entry per table: ``name``, ``role``, ``description``, ``hidden``,
       ``sample_rows``, and ``columns`` (each with ``name``, ``dataType``,
       ``key``, ``hidden``, ``format``, ``description``). The calendar
       entry also carries its ``range``.
   * - ``relationships``
     - ``from`` (many side), ``to`` (one side), ``active``, and
       ``crossFilter`` (``single`` or ``both``).
   * - ``measures``
     - ``name``, ``table``, ``expression``, ``format``, ``displayFolder``,
       ``hidden``, ``description``, and ``dependsOn``: the ``measures``,
       ``columns`` and ``tables`` the DAX refers to.

A measure entry from the hospital example:

::

    {
      "dependsOn": {
        "columns": [],
        "measures": ["Patients #", "Total Billed Revenue"],
        "tables": []
      },
      "description": "Average billed amount per distinct patient seen.",
      "displayFolder": "Ratios",
      "expression": "DIVIDE([Total Billed Revenue], [Patients #])",
      "format": "$ #,0.00",
      "hidden": false,
      "name": "Avg Billed per Patient",
      "table": "Clinic Measures"
    }

Table roles
-------------

The role comes from the spec. A table on the "one" side of any relationship
is a dimension; a table only on the "many" side is a fact; the calendar and
the measures table have their own roles. A spec can set ``"role"`` on a
table to override the inference.
