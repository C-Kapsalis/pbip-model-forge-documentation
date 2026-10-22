Build a model from a prompt
===============================

This guide uses the two Claude Code skills. It assumes you have installed
both tools and cloned the tool's repository (:doc:`install`), and that
Claude Code is open in the clone.

The two skills, called with a real domain
------------------------------------------------

Ask for a model, describing the business domain in your own words:

::

    /create-sample-model create a sample data model for a hospital

The skill writes a spec and builds a ``.pbip`` folder from it: a hidden
fact table at the grain of the domain, two to four dimension tables, a
calendar, a measures table, and sample rows already loaded. It shows you
the result once ``pbip-forge check`` reports no P0 or P1 violations and
``tmdl-preflight`` reports no errors. Next to the model sits a hand-off
pack, ``handoff/SPEC.md`` and ``handoff/manifest.json``, describing every
table, relationship and measure; see :doc:`/reference/handoff-pack`.

Then grow it, describing what is missing the same way:

::

    /validate-and-expand add an insurance company entity and measures
    relating it to relevant existing entities

The model comes back rebuilt from the changed spec: a new table for the
insurer, a relationship to the existing patients, and new measures,
checked by the same two gates.

The repository keeps this hospital model as ``scripts/build_hospital.py``,
and the next two guides build on it. To build that copy yourself, from
the root of the clone:

::

    pbip-forge build scripts/build_hospital.py out/Hospital --check --profile handoff

::

    Built PBIP: out/Hospital/Hospital.pbip

Other prompts that work the same way
------------------------------------------

The pattern is the same for any domain. A few more to try:

- ``/create-sample-model create a sample data model for subscription analytics``
- ``/create-sample-model create a sample data model for a logistics company``
- ``/create-sample-model create a sample data model for a bakery``

Each produces its own star schema, shaped by the domain described, with
its own sample rows.

The result, opened in Power BI Desktop
--------------------------------------------

.. figure:: /_static/screenshots/hospital-model-open.png
   :alt: A hospital data model open in Power BI Desktop's model view. Departments, Physicians, Diagnoses and Patients each connect to the hidden Encounters fact table; Patients also connects to InsuranceCompanies, whose key is company_id; __Calendar connects to Encounters by two date relationships. A Clinic Measures table lists Avg Billed per Encounter, Avg Billed per Patient, Avg Length of Stay, Encounters #, Insured Patients #, Revenue Share by Insurer, Total Billed Revenue and Total Patient Days.

   A hospital model in Power BI Desktop's model view, with the insurance
   company entity in place. The screenshot was taken with an earlier
   version of the example: there the insurer's key is ``company_id`` and
   two measures are ``Insured Patients #`` and ``Revenue Share by
   Insurer``. In ``scripts/build_hospital.py`` at 1.2.1 the key is
   ``insurance_company_id``, and the measures are ``Patients #`` and
   ``Insurer Revenue Share %``.

Where to go next
--------------------

The hospital model is what the next two guides build on, one visual
each, to read the numbers it already contains:

- :doc:`build-a-line-and-clustered-column-chart`
- :doc:`build-a-drill-down-matrix`
