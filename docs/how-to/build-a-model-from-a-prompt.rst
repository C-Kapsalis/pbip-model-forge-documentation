Build a model from a prompt
===============================

The two skills, called with a real domain
------------------------------------------------

Ask for a model, describing the business domain in your own words:

::

    /create-sample-model create a sample data model for a hospital

That builds a working ``.pbip`` file: a hidden fact table at the
right grain, two to four dimension tables, a calendar, a measures
table, and believable sample rows already loaded, confirmed to open
cleanly before you see it.

Then grow it, describing what is missing the same way:

::

    /validate-and-expand add an insurance company entity and measures
    relating it to relevant existing entities

The model comes back updated: a new table for the insurer, a
relationship into the existing admissions, and one or two new
measures, re-validated the same way the original build was.

Other prompts that work the same way
------------------------------------------

The pattern is the same regardless of domain. A few more to try:

- ``/create-sample-model create a sample data model for subscription analytics``
- ``/create-sample-model create a sample data model for a logistics company``
- ``/create-sample-model create a sample data model for a bakery``

Each produces its own star schema, shaped by the domain described,
with its own believable sample rows.

The result, opened in Power BI Desktop
--------------------------------------------

.. Screenshot to take: the hospital model above, opened in Power BI
   Desktop, showing the model/diagram view with its tables and
   relationships visible. Save as
   docs/_static/screenshots/hospital-model-open.png.

.. image:: /_static/screenshots/hospital-model-open.png
   :alt: The generated hospital data model open in Power BI Desktop's model view, showing the fact table and its dimension tables connected by relationships.

Where to go next
--------------------

The hospital model above is what the next two how-to guides build on,
one visual each, to read the numbers it already contains:

- :doc:`build-a-line-and-clustered-column-chart`
- :doc:`build-a-drill-down-matrix`
