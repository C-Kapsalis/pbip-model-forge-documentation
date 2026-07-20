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
relationship into the existing patient encounters, and a handful of
new measures, re-validated the same way the original build was.

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

.. figure:: /_static/screenshots/hospital-model-open.png
   :alt: The generated hospital data model open in Power BI Desktop's model view. Departments, Physicians, Diagnoses, and Patients each connect into the hidden Encounters fact table; Patients also connects to InsuranceCompanies; a Calendar table connects by date. A Clinic Measures table lists Total Billed Revenue, Avg Billed per Encounter, Avg Billed per Patient, Avg Length of Stay, Encounters #, Insured Patients #, Revenue Share by Insurer, and Total Patient Days.

   The generated hospital model, every table already connected and the
   insurance company entity in place.

Where to go next
--------------------

The hospital model above is what the next two how-to guides build on,
one visual each, to read the numbers it already contains:

- :doc:`build-a-line-and-clustered-column-chart`
- :doc:`build-a-drill-down-matrix`
