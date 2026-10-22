pbip-forge check rules
========================

``pbip-forge check`` judges a model's spec, sample rows included, before any
file is written. ``tmdl-preflight`` answers "will Power BI Desktop open
this?"; ``pbip-forge check`` answers "is this a model a data engineer can
trust?". Both skills run it first and stop until it reports no P0 or P1
violations.

Running it
------------

The spec is either a JSON file or a Python file that defines a module-level
``SPEC`` dict, like the example scripts in the repository:

::

    pbip-forge check scripts/build_hospital.py --profile handoff
    pbip-forge check spec.json --format json
    pbip-forge rules

The command prints one line per violation, its fix, and a summary. A spec
whose ratio counts a dimension table instead of the fact's foreign key
comes back like this:

::

    P1 FM015 'Clinic Measures'[Avg Billed per Patient]: DIVIDE denominator counts dimension table 'Patients', so filters from other dimensions do not reach it
        fix: Count from the fact instead: DISTINCTCOUNT(Encounters[patient_id]).
    pbip-forge check (handoff): 0 P0, 1 P1, 0 P2

A spec that passes prints only the summary line:

::

    pbip-forge check (handoff): 0 P0, 0 P1, 0 P2

It exits with status 1 when any P0 or P1 violation is found, and 0
otherwise. From Python, ``build_pbip(spec, out_dir, check=True,
profile="handoff")`` runs the same check and raises ``SpecCheckError``
without writing anything when it fails.

``check`` is one of eight commands. The other seven — ``build``,
``rules``, ``table dump``, ``report blank``, ``clone-report``,
``f-filters`` and ``i-filters`` — are in :doc:`cli`.

Severities and profiles
-------------------------

.. list-table::
   :widths: 10 90
   :header-rows: 1

   * - Severity
     - Meaning
   * - P0
     - The model is broken or computes wrong numbers.
   * - P1
     - Must be fixed before the model is handed off.
   * - P2
     - Advisory.

Two profiles set how strict the documentation and naming rules are:

- ``mockup`` (the default) treats them as advisory (P2), for quick drafts.
- ``handoff`` makes them P1, for a model you are about to hand to engineers.

Structural, data and DAX rules have the same severity in both profiles. The
gate is the same in both: **0 P0 and 0 P1**.

Rule catalog
--------------

.. list-table::
   :widths: 12 30 12 12
   :header-rows: 1

   * - Rule
     - Name
     - mockup
     - handoff
   * - :ref:`FM001 <fm001>`
     - relationship-endpoints
     - P0
     - P0
   * - :ref:`FM002 <fm002>`
     - row-width
     - P0
     - P0
   * - :ref:`FM003 <fm003>`
     - dimension-single-key
     - P1
     - P1
   * - :ref:`FM004 <fm004>`
     - key-unique-non-null
     - P0
     - P0
   * - :ref:`FM005 <fm005>`
     - fk-orphans
     - P1
     - P1
   * - :ref:`FM006 <fm006>`
     - key-type-match
     - P0
     - P0
   * - :ref:`FM007 <fm007>`
     - key-name-match
     - P2
     - P2
   * - :ref:`FM008 <fm008>`
     - fact-calendar
     - P1
     - P1
   * - :ref:`FM009 <fm009>`
     - dimension-connected
     - P1
     - P1
   * - :ref:`FM010 <fm010>`
     - no-bidirectional
     - P1
     - P1
   * - :ref:`FM011 <fm011>`
     - single-active-path
     - P0
     - P0
   * - :ref:`FM012 <fm012>`
     - no-relationship-cycle
     - P0
     - P0
   * - :ref:`FM013 <fm013>`
     - dax-references
     - P0
     - P0
   * - :ref:`FM014 <fm014>`
     - fact-read-by-measure
     - P1
     - P1
   * - :ref:`FM015 <fm015>`
     - ratio-grain
     - P1
     - P1
   * - :ref:`FM016 <fm016>`
     - measure-format
     - P2
     - P1
   * - :ref:`FM017 <fm017>`
     - measure-description
     - P2
     - P1
   * - :ref:`FM018 <fm018>`
     - measure-display-folder
     - P2
     - P1
   * - :ref:`FM019 <fm019>`
     - name-format-agreement
     - P2
     - P1
   * - :ref:`FM020 <fm020>`
     - fact-dates-in-calendar
     - P1
     - P1
   * - :ref:`FM021 <fm021>`
     - table-description
     - P2
     - P1
   * - :ref:`FM022 <fm022>`
     - column-description
     - P2
     - P2

.. _fm001:

FM001: relationship-endpoints
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**Severity:** P0 in ``mockup``, P0 in ``handoff``.

**Why:** A relationship whose table or column does not exist cannot be built.

**Fix:** Correct the table or column name in the relationship, or add the column.

.. _fm002:

FM002: row-width
~~~~~~~~~~~~~~~~

**Severity:** P0 in ``mockup``, P0 in ``handoff``.

**Why:** Every sample row must have one value per declared column, or the entered-data partition loads shifted or missing values.

**Fix:** Add or remove values so each row matches the column list.

.. _fm003:

FM003: dimension-single-key
~~~~~~~~~~~~~~~~~~~~~~~~~~~

**Severity:** P1 in ``mockup``, P1 in ``handoff``.

**Why:** A dimension needs exactly one key column for relationships to land on and for engineers to know its grain.

**Fix:** Mark exactly one column with "key": True.

.. _fm004:

FM004: key-unique-non-null
~~~~~~~~~~~~~~~~~~~~~~~~~~

**Severity:** P0 in ``mockup``, P0 in ``handoff``.

**Why:** The one side of a relationship must be unique and non-blank, or the relationship fails to refresh.

**Fix:** Make every key value distinct and non-null.

.. _fm005:

FM005: fk-orphans
~~~~~~~~~~~~~~~~~

**Severity:** P1 in ``mockup``, P1 in ``handoff``.

**Why:** Fact rows whose foreign key matches no dimension key (or is null) land on the blank member and silently drop out of sliced totals.

**Fix:** Add an 'Unknown' row (for example key -1) to the dimension and point unmatched or null foreign keys at it.

.. _fm006:

FM006: key-type-match
~~~~~~~~~~~~~~~~~~~~~

**Severity:** P0 in ``mockup``, P0 in ``handoff``.

**Why:** Relationship endpoints of different data types never match.

**Fix:** Give the foreign key the same type as the key it points to.

.. _fm007:

FM007: key-name-match
~~~~~~~~~~~~~~~~~~~~~

**Severity:** P2 in ``mockup``, P2 in ``handoff``.

**Why:** A foreign key named exactly like its primary key makes the join obvious to everyone who reads the model.

**Fix:** Rename one side so the FK and PK share a name.

.. _fm008:

FM008: fact-calendar
~~~~~~~~~~~~~~~~~~~~

**Severity:** P1 in ``mockup``, P1 in ``handoff``.

**Why:** A fact without an active relationship to the calendar cannot be sliced by date.

**Fix:** Add an active relationship from the fact's primary date column to __Calendar.Date (add a calendar to the spec if there is none).

.. _fm009:

FM009: dimension-connected
~~~~~~~~~~~~~~~~~~~~~~~~~~

**Severity:** P1 in ``mockup``, P1 in ``handoff``.

**Why:** A dimension that reaches no fact filters nothing; it is either dead weight or a missing relationship.

**Fix:** Relate it to a fact (directly or through another dimension), or remove it.

.. _fm010:

FM010: no-bidirectional
~~~~~~~~~~~~~~~~~~~~~~~

**Severity:** P1 in ``mockup``, P1 in ``handoff``.

**Why:** Bidirectional filters create ambiguous paths and surprising totals.

**Fix:** Use single-direction filtering; express the cross-filter in a measure (CROSSFILTER or TREATAS) where it is really needed.

.. _fm011:

FM011: single-active-path
~~~~~~~~~~~~~~~~~~~~~~~~~

**Severity:** P0 in ``mockup``, P0 in ``handoff``.

**Why:** Two active paths between the same tables make filter propagation ambiguous; Power BI deactivates one of them on its own.

**Fix:** Mark all but one path "active": False and use USERELATIONSHIP in the measures that need the other.

.. _fm012:

FM012: no-relationship-cycle
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**Severity:** P0 in ``mockup``, P0 in ``handoff``.

**Why:** A relationship cycle (A filters B filters ... filters A) cannot be resolved by the engine.

**Fix:** Remove or reverse one relationship in the cycle.

.. _fm013:

FM013: dax-references
~~~~~~~~~~~~~~~~~~~~~

**Severity:** P0 in ``mockup``, P0 in ``handoff``.

**Why:** A DAX reference to a table, column or measure that does not exist makes the measure fail.

**Fix:** Fix the name, or add the missing column or measure.

.. _fm014:

FM014: fact-read-by-measure
~~~~~~~~~~~~~~~~~~~~~~~~~~~

**Severity:** P1 in ``mockup``, P1 in ``handoff``.

**Why:** A fact no measure reads is dead weight, or the measures that should summarize it are missing. Filter sensors (ISFILTERED, ISEMPTY, ISINSCOPE, HASONEVALUE, ...) do not count as reads.

**Fix:** Add at least one measure that aggregates the fact.

.. _fm015:

FM015: ratio-grain
~~~~~~~~~~~~~~~~~~

**Severity:** P1 in ``mockup``, P1 in ``handoff``.

**Why:** A ratio whose denominator counts a dimension table ignores filters from the other dimensions (they filter the fact, not that dimension), so per-slice values are wrong.

**Fix:** Count the entity from the fact's foreign key instead, for example DISTINCTCOUNT(Fact[fk]).

.. _fm016:

FM016: measure-format
~~~~~~~~~~~~~~~~~~~~~

**Severity:** P2 in ``mockup``, P1 in ``handoff``.

**Why:** A visible measure without a format string shows raw engine formatting (long decimals, no currency).

**Fix:** Set "format" on the measure.

.. _fm017:

FM017: measure-description
~~~~~~~~~~~~~~~~~~~~~~~~~~

**Severity:** P2 in ``mockup``, P1 in ``handoff``.

**Why:** Consumers read the description as the measure's definition; an empty or placeholder one leaves it ambiguous.

**Fix:** Write a one-line description of at least 10 characters (no TBD/TODO).

.. _fm018:

FM018: measure-display-folder
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**Severity:** P2 in ``mockup``, P1 in ``handoff``.

**Why:** Display folders keep the field list navigable; the builder defaults to Counts / Ratios / Base, so this fires only on an explicit empty folder.

**Fix:** Remove the empty "displayFolder" or name a folder.

.. _fm019:

FM019: name-format-agreement
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**Severity:** P2 in ``mockup``, P1 in ``handoff``.

**Why:** The naming convention promises the format: 'X %' is a percentage, 'X #' is a whole-number count.

**Fix:** Use a percent format for '%' names and a whole-number format such as #,0 for '#' names (or rename the measure).

.. _fm020:

FM020: fact-dates-in-calendar
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**Severity:** P1 in ``mockup``, P1 in ``handoff``.

**Why:** Fact dates outside the calendar range join to nothing and vanish from date-sliced visuals.

**Fix:** Widen the calendar's start_year/end_year, or omit them so they are inferred from the data.

.. _fm021:

FM021: table-description
~~~~~~~~~~~~~~~~~~~~~~~~

**Severity:** P2 in ``mockup``, P1 in ``handoff``.

**Why:** A one-line table description tells engineers the grain and purpose.

**Fix:** Set "description" on the table, e.g. 'One row per order line.'

.. _fm022:

FM022: column-description
~~~~~~~~~~~~~~~~~~~~~~~~~

**Severity:** P2 in ``mockup``, P2 in ``handoff``.

**Why:** Visible columns are what report authors see; a description saves a trip to the source system.

**Fix:** Set "description" on each visible column.

