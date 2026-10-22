Prototype a model from a spec file
=====================================

This guide takes you from a model you have in mind to a PBIP folder and a
hand-off pack your data engineers can build from. You write the spec
yourself, as a JSON file; no Claude Code session is needed. The example
is a small, invented bike shop: two dimensions, one fact, a calendar and
four measures.

Before you start, install ``pbip-forge`` and ``tmdl-preflight`` 1.2.1
(:doc:`install`). Every command below is the same in PowerShell and in
bash.

1. Write the spec
-------------------

Make a folder for the work and open a terminal in it:

::

    mkdir bikeshop
    cd bikeshop

Save this as ``bikeshop-spec.json`` in that folder, as UTF-8 without a
byte order mark. A spec file that starts with a byte order mark stops
``pbip-forge`` with a Python traceback ending in
``json.decoder.JSONDecodeError: Unexpected UTF-8 BOM``.

.. code-block:: json

    {
      "name": "BikeShop",
      "tables": [
        {
          "name": "Products",
          "description": "One row per product the shop sells.",
          "columns": [
            {"name": "product_id", "type": "int64", "key": true},
            {"name": "product_name", "type": "string", "description": "Product display name."},
            {"name": "category", "type": "string", "description": "Product category, such as Bikes or Parts."}
          ],
          "rows": [
            [-1, "Unknown", "Unknown"],
            [1, "Road Bike", "Bikes"],
            [2, "Trail Bike", "Bikes"],
            [3, "Helmet", "Parts"],
            [4, "Inner Tube", "Parts"]
          ]
        },
        {
          "name": "Stores",
          "description": "One row per store.",
          "columns": [
            {"name": "store_id", "type": "int64", "key": true},
            {"name": "store_name", "type": "string", "description": "Store display name."},
            {"name": "city", "type": "string", "description": "City the store is in."}
          ],
          "rows": [
            [-1, "Unknown", "Unknown"],
            [1, "Harbor Street", "Northport"],
            [2, "Hill Road", "Eastvale"]
          ]
        },
        {
          "name": "Sales",
          "description": "One row per order line.",
          "hidden": true,
          "columns": [
            {"name": "order_date", "type": "datetime"},
            {"name": "product_id", "type": "int64"},
            {"name": "store_id", "type": "int64"},
            {"name": "quantity", "type": "int64", "summarizeBy": "sum"},
            {"name": "amount", "type": "double", "summarizeBy": "sum"}
          ],
          "rows": [
            ["2024-01-15", 1, 1, 1, 1200.0],
            ["2024-01-20", 3, 1, 2, 90.0],
            ["2024-02-03", 2, 2, 1, 1500.0],
            ["2024-02-11", 4, 2, 5, 40.0],
            ["2024-03-02", 3, 2, 1, 45.0],
            ["2024-03-09", 1, 1, 1, 1200.0]
          ]
        }
      ],
      "calendar": {"name": "__Calendar"},
      "measures_table": {
        "name": "Shop Measures",
        "description": "Home table for every measure.",
        "measures": [
          {"name": "Total Sales", "expression": "SUM(Sales[amount])",
           "format": "$ #,0.00", "description": "Total sales amount."},
          {"name": "Units Sold #", "expression": "SUM(Sales[quantity])",
           "format": "#,0", "description": "Number of units sold."},
          {"name": "Stores #", "expression": "DISTINCTCOUNT(Sales[store_id])",
           "format": "#,0", "description": "Number of distinct stores."},
          {"name": "Avg Sales per Store", "expression": "DIVIDE([Total Sales], [Stores #])",
           "format": "$ #,0.00", "description": "Average sales amount per store."}
        ]
      },
      "relationships": [
        {"from": "Sales.product_id", "to": "Products.product_id"},
        {"from": "Sales.store_id", "to": "Stores.store_id"},
        {"from": "Sales.order_date", "to": "__Calendar.Date"}
      ]
    }

What the spec says:

- ``Sales`` is the fact: one row per order line, hidden, with a foreign
  key to each dimension and a date.
- Each dimension has one key column and an "Unknown" row with key ``-1``,
  for fact rows that have no real match.
- ``Stores #`` counts stores from the fact's foreign key,
  ``Sales[store_id]``, so any filter that reaches the fact also reaches
  the count.
- The calendar's year range is taken from the order dates.

The sample rows are written into the model files, so invent them. Do not
copy real customer records into a spec. Every key is described in
:doc:`../reference/spec-format`.

2. Check the spec
-------------------

::

    pbip-forge check bikeshop-spec.json --profile handoff

A spec ready to hand off prints one line and exits 0:

::

    pbip-forge check (handoff): 0 P0, 0 P1, 0 P2

The gate is 0 P0 and 0 P1. ``--profile handoff`` makes five
documentation and naming rules P1 (measure formats, measure
descriptions, display folders, name and format agreement, table
descriptions); the default ``mockup`` profile treats them as advisory
P2.

When the check finds a problem it names the rule, the object and the fix,
and exits 1. If ``Stores #`` counted the dimension's own key,
``DISTINCTCOUNT(Stores[store_id])``, the check would print:

::

    P1 FM015 'Shop Measures'[Avg Sales per Store]: DIVIDE denominator counts dimension table 'Stores', so filters from other dimensions do not reach it
        fix: Count from the fact instead: DISTINCTCOUNT(Sales[store_id]).
    pbip-forge check (handoff): 0 P0, 1 P1, 0 P2

Change the spec as the fix says and run the check again.
:doc:`../reference/check-rules` lists all 22 rules.

3. Build the model
--------------------

::

    pbip-forge build bikeshop-spec.json out/BikeShop --check --profile handoff

``--check`` runs the same check first and writes nothing if it fails. If
``out/BikeShop`` already exists, the build overwrites what it wrote there
before; `Change the model`_ explains what that touches. A successful
build prints one line:

::

    Built PBIP: out/BikeShop/BikeShop.pbip

The folder it writes:

::

    out/BikeShop/
        BikeShop.pbip
        BikeShop.SemanticModel/
        BikeShop.Report/
        handoff/
            SPEC.md
            manifest.json

The semantic model holds the five tables (``Products``, ``Stores``,
``Sales``, ``__Calendar`` and ``Shop Measures``), the three relationships
and the four measures, with the sample rows loaded. The report holds one
empty page, ``Page 1``, and no visuals. :doc:`../reference/handoff-pack`
lists every file.

4. Validate the files
-----------------------

::

    tmdl-preflight check out/BikeShop

The gate is 0 errors. The first and last lines of the run:

::

    BikeShop.SemanticModel/definition/tables/__Calendar.tmdl  R013 info: calendar __Calendar has no Year, Month, Week, Day column(s). [__Calendar]

    scanned: 1 model, 5 tables, 22 columns, 4 measures, 3 relationships, 5 DAX expressions, 1 report
    ...
    tmdl-preflight: 0 error(s), 0 warning(s), 1 info(s)

Between those lines, tmdl-preflight prints a summary for each of its
relationship and calendar rules. The one info, R013, is advisory: the
generated calendar names its columns ``Calendar Year``, ``Calendar Month``
and so on, not ``Year`` and ``Month``. The command exits 0 when there are
no errors.

5. Hand it to the engineers
-----------------------------

Send the spec file and the whole ``out/BikeShop`` folder. In it:

- ``handoff/SPEC.md`` is the page to read first: a Mermaid diagram of
  every table and relationship, a data dictionary, the relationships and
  every measure with its DAX.
- ``handoff/manifest.json`` is the same model in machine-readable form,
  with the tables, columns, relationships and each measure's
  dependencies.
- ``BikeShop.pbip`` and its two folders are the model itself, with the
  sample data loaded.

The top of ``SPEC.md`` for this spec:

::

    # BikeShop: model spec

    Generated by pbip-model-forge 1.2.1 from the model's spec dict. Regenerate it rather than editing it by hand.

    ## Diagram

    ```mermaid
    erDiagram
        __Calendar ||--o{ Sales : "order_date"
        Products ||--o{ Sales : "product_id"
        Stores ||--o{ Sales : "store_id"

Write down, next to the files, what the spec cannot tell the engineers:
which questions each measure answers, where each table's data lives
today, how often it must refresh, how much history and how many rows to
expect, and who may see which rows. Copy the last line of each gate run
into the note (``pbip-forge check (handoff): 0 P0, 0 P1, 0 P2`` and
``tmdl-preflight: 0 error(s), ...``), and say that opening the model in
Power BI Desktop is a manual step: the two tools check the files, and
neither one opens them in Desktop.

6. Open it in Power BI Desktop
--------------------------------

On Windows, open ``out/BikeShop/BikeShop.pbip`` in Power BI Desktop to look
at the model and try the measures on the empty page. This is the one step
the tools cannot do for you.

Change the model
------------------

Edit the spec and run steps 2 to 4 again. Do not edit the files in
``out/BikeShop`` by hand: the next build overwrites them.

A build into an existing folder deletes every ``.tmdl`` file in the
model's ``definition/tables`` folder, then rewrites every file it writes,
the report's ``pages.json`` and ``report.json`` included. Other files
stay where they are. Keep report work you want to keep in a separate
report folder, not in the one the build writes.
