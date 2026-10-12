Skills
========

pbip-model-forge is used through a Claude Code skill pair: everything happens
through two slash commands typed directly into a Claude Code conversation. Each
command runs two gates internally before you see the result:
:doc:`pbip-forge check <check-rules>` must report no P0 or P1 violations, and
then tmdl-preflight must report zero errors. Every model these skills produce
is therefore both modeled to best practice and confirmed to open cleanly in
Power BI Desktop.

.. list-table::
   :widths: 20 40 40
   :header-rows: 1

   * - skill
     - what you give it
     - what you get back
   * - `/create-sample-model`
     - a business domain, described in your own words
     - a working .pbip file, plus its generated SPEC.md: diagram and data dictionary
   * - `/validate-and-expand`
     - an existing model, plus what you want added or checked
     - the same model, updated and re-validated, still openable in Power BI Desktop

``/create-sample-model``
----------------------------

Give it one sentence describing a business domain, for example "create a
sample data model for a veterinary clinic." What comes back:

- **A hidden fact table** at the right grain for the domain (one row per
  visit, per encounter, per order, whatever the domain's core recurring
  event is), with foreign keys into every dimension and a date column.
  Each foreign key is named exactly like the key it points to, and every
  value has a match; a fact row with no real match points at an
  "Unknown" member instead of being left blank.
- **A handful of dimension tables**, each with exactly one key column and
  a few descriptive attributes. For the veterinary clinic: Patients (each
  animal's name, species, breed, and owner), Veterinarians (name and
  specialty), and Services (a billable service catalog with prices).
- **A calendar table** (year, quarter, month, week and day-of-week
  columns, its range inferred from the data) and **a measures table** with
  several DAX measures already defined, formatted, described and filed
  into display folders, for the clinic: Total Revenue, Visits #,
  Patients #, Avg Cost per Visit, and Total Visit Hours. Measure names
  follow one convention: ``X #`` for counts, ``X %`` for shares, ``Avg X``,
  ``Total X``, and ``X per Y``. Distinct counts of an entity, such as
  patients, are always taken from the fact table's foreign key, so they
  stay correct however the report is sliced.
- **Believable sample rows already loaded** in every table, enough that
  the measures compute to real numbers immediately, spanning a plausible
  date range (the clinic model's visits ran from early 2024 through late
  2025).
- **A hand-off pack** next to the model: ``handoff/SPEC.md``, with a
  diagram of every table and relationship and a data dictionary, and
  ``handoff/manifest.json``, the same model in machine-readable form. See
  :doc:`handoff-pack`.
- **Confirmation the model is clean**: no P0 or P1 violations from
  ``pbip-forge check``, and zero errors from tmdl-preflight, before it is
  ever shown to you.

``/validate-and-expand``
----------------------------

Give it an existing model plus what it is missing, in plain language, for
example "add a pet insurance company entity and measures relating it to
relevant existing entities." What comes back:

- **A new table**, when the request calls for one, at whatever grain
  fits. The veterinary example added an InsuranceProviders table: four
  real insurers plus a "Self-Pay" entry, each with a plan type and a
  coverage rate.
- **New fields on the existing fact table**, wired to the new entity: the
  clinic's Visits table gained a foreign key to the insurer, plus how
  much was claimed and how much was actually covered, per visit.
- **New measures**, reachable the same way the original ones are, by
  patient, by vet, by service, or by date. The insurance expansion added
  six: Total Claims Submitted, Total Covered Amount, Total Out-of-Pocket
  Revenue, Insurance Reimbursement %, Insured Visits #, and Insured
  Visits %.
- **A refreshed hand-off pack and a re-validation**, the same things the
  first build produced, confirming the change fits cleanly: still no P0
  or P1 violations, still zero errors.
- **The original tables, untouched.** Nothing about the patients, the
  vets, or the services needed to change for the new entity to fit in
  alongside them.

Related topics
--------------

* :doc:`/topic-guides` explains what an ontology means for
  `/create-sample-model`, and how `/validate-and-expand` fits a new
  entity into one that already exists.
