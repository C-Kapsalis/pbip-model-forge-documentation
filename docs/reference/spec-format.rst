The spec format
=================

A spec is one object that describes the whole model: its tables, their
columns and sample rows, the calendar, the measures and the relationships.
``pbip-forge check`` judges it and ``pbip-forge build`` writes the model
from it. The model is never edited by hand: change the spec, check it and
build again.

The spec is either a JSON file (UTF-8, without a byte order mark) or a
Python file that defines a
module-level ``SPEC`` dict with the same keys. The JSON form is the one
to use when you write a spec yourself; the Python form is what the
example scripts in the tool's repository use
(``scripts/build_vet_clinic.py``, ``scripts/build_hospital.py``).
:doc:`../how-to/build-a-model-from-a-spec` walks through a complete JSON
spec.

Top level
-----------

.. list-table::
   :widths: 20 12 68
   :header-rows: 1

   * - Key
     - Required
     - Meaning
   * - ``name``
     - yes
     - The model name. It names the ``.pbip`` file and the
       ``<name>.SemanticModel`` and ``<name>.Report`` folders.
   * - ``tables``
     - yes
     - The data tables: dimensions and facts. See `Tables`_.
   * - ``calendar``
     - no
     - The generated date table. See `Calendar`_.
   * - ``measures_table``
     - no
     - The table that holds every measure. See `Measures`_.
   * - ``relationships``
     - no
     - The relationships between tables. See `Relationships`_.

Tables
--------

.. list-table::
   :widths: 20 12 68
   :header-rows: 1

   * - Key
     - Required
     - Meaning
   * - ``name``
     - yes
     - The table name.
   * - ``columns``
     - yes
     - The column list, in the order the values appear in each row.
   * - ``rows``
     - no
     - The sample rows: one list per row, one value per column. Dates are
       written as ``"YYYY-MM-DD"`` text in JSON.
   * - ``description``
     - no
     - One line on the table's grain and purpose, for example ``"One row
       per order line."``. Rule FM021 makes it required in the ``handoff``
       profile.
   * - ``hidden``
     - no
     - ``true`` hides the table. A fact table is usually hidden, so report
       authors use the measures instead of its raw columns.
   * - ``role``
     - no
     - ``fact`` or ``dimension``. Without it the role is inferred: a table
       on the ``to`` side of any relationship is a dimension, a table only
       on the ``from`` side is a fact, and a table in no relationship is a
       dimension.

Columns
~~~~~~~~~

.. list-table::
   :widths: 20 12 68
   :header-rows: 1

   * - Key
     - Required
     - Meaning
   * - ``name``
     - yes
     - The column name.
   * - ``type``
     - yes
     - One of ``int64`` (or ``int``), ``string`` (or ``text``),
       ``double`` (or ``number``), ``decimal``, ``datetime`` (or ``date``),
       ``boolean`` (or ``bool``).
   * - ``key``
     - no
     - ``true`` marks the dimension's key column. A dimension needs exactly
       one (rule FM003).
   * - ``description``
     - no
     - One line for report authors. Rule FM022 reports a visible column
       without one as advisory (P2) in both profiles.
   * - ``format``
     - no
     - A Power BI format string, such as ``"$ #,0.00"``.
   * - ``summarizeBy``
     - no
     - The default aggregation, such as ``"sum"``. The default is ``none``.
   * - ``hidden``
     - no
     - ``true`` or ``false``. It overrides the builder's default, below.
   * - ``isAvailableInMdx``
     - no
     - ``true`` or ``false``. It overrides the builder's default, below.

The builder hides three kinds of column unless the spec says otherwise:
key columns, the ``from`` column of every relationship, and the numeric
columns of a fact table that are neither keys nor relationship columns
(the raw values the measures add up). That last kind also gets
``isAvailableInMdx: false``.

``pbip-forge check`` does not validate ``type``. A type outside the list
above passes the check and stops ``build`` with an error that lists the
supported types:

::

    ValueError: unknown column type 'varchar'; supported: ['bool', 'boolean', 'date', 'datetime', 'decimal', 'double', 'int', 'int64', 'number', 'string', 'text']

Calendar
----------

::

    "calendar": {"name": "__Calendar", "start_year": 2024, "end_year": 2025}

``build`` writes a calculated date table with these columns: ``Date``,
``Calendar Year``, ``Calendar Quarter``, ``Quarter``, ``Calendar Month``,
``Calendar Month Name``, ``Month Start``, ``Year-Month``, ``Week Start``,
``Day of Week Number`` and ``Day of Week``. The three number columns that
sort another column (``Calendar Quarter``, ``Calendar Month`` and ``Day of
Week Number``) are hidden. The table also gets a ``Date Hierarchy`` with
the levels ``Calendar Year``, ``Quarter``, ``Calendar Month Name`` and
``Date``.

``start_year`` and ``end_year`` are optional: without them, the range is
taken from the date columns of the fact tables. Give the ``name`` key
every time. An empty object, ``"calendar": {}``, counts as no calendar:
``check`` then reports FM008 (``the spec has no calendar``) on every fact.

A fact table reaches the calendar through an ordinary relationship to
``<calendar name>.Date``.

Measures
----------

::

    "measures_table": {
      "name": "Shop Measures",
      "description": "Home table for every measure.",
      "measures": [
        {"name": "Total Sales", "expression": "SUM(Sales[amount])",
         "format": "$ #,0.00", "description": "Total sales amount."}
      ]
    }

The table needs a ``name``; ``description`` is optional. Do not name any
table ``Measures``, in any letter case: ``check`` lets the name through,
but ``build`` stops with this error and writes no tables:

::

    ValueError: table name 'Measures' is reserved in Power BI; Desktop refuses to open the model ('Unsupported Table name'). Rename it, e.g. 'Key Measures' or '<Domain> Measures'.

Each measure takes these keys:

.. list-table::
   :widths: 20 12 68
   :header-rows: 1

   * - Key
     - Required
     - Meaning
   * - ``name``
     - yes
     - The measure name. The names follow one convention: ``X #`` for
       counts, ``X %`` for shares, ``Avg X``, ``Total X`` and ``X per Y``.
       Rule FM019 checks that a ``#`` name has a whole-number format and a
       ``%`` name a percent format.
   * - ``expression``
     - yes
     - The DAX. Every table, column and measure it names must exist (rule
       FM013).
   * - ``format``
     - no
     - The format string. Required for a visible measure in the
       ``handoff`` profile (rule FM016).
   * - ``description``
     - no
     - One line, at least 10 characters. Required in the ``handoff``
       profile (rule FM017).
   * - ``displayFolder``
     - no
     - The folder in the field list. Without it the builder picks
       ``Counts``, ``Ratios`` or ``Base`` from the name and the DAX.
   * - ``helper``
     - no
     - ``true`` hides the measure and files it in the ``_Helpers``
       folder.
   * - ``hidden``
     - no
     - ``true`` or ``false``. It overrides the default (hidden only when
       ``helper`` is ``true``).

Write a ratio as ``DIVIDE`` of two base measures, and count an entity from
the fact table's foreign key, ``DISTINCTCOUNT(Sales[store_id])``, not from
the dimension's own key. Rule FM015 reports the second mistake;
:ref:`fm015` explains why it gives wrong numbers.

Relationships
---------------

::

    "relationships": [
      {"from": "Sales.product_id", "to": "Products.product_id"},
      {"from": "Sales.order_date", "to": "__Calendar.Date"}
    ]

.. list-table::
   :widths: 20 12 68
   :header-rows: 1

   * - Key
     - Required
     - Meaning
   * - ``from``
     - yes
     - ``Table.column`` on the many side: the fact's foreign key.
   * - ``to``
     - yes
     - ``Table.column`` on the one side: the dimension's key.
   * - ``active``
     - no
     - ``false`` makes the relationship inactive. The default is
       ``true``. Two active paths between the same tables are a P0 error
       (rule FM011).
   * - ``crossFilter``
     - no
     - ``both`` makes the filter run in both directions. Rule FM010
       reports it as P1 in both profiles; leave the key out for single
       direction.

Name a foreign key exactly like the key it points to (rule FM007), and
give every dimension an "Unknown" row, key ``-1``, for fact rows that
have no real match (rule FM005 reports a fact value with no match).
