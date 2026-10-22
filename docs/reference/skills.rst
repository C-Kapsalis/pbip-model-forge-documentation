Skills
========

pbip-model-forge ships two Claude Code skills, in the tool's repository
under ``.claude/skills``. Open a Claude Code session in a clone of the
repository (:doc:`../how-to/install`) and call them as slash commands, or
describe what you want and let Claude Code pick the skill. Each skill
writes the spec, builds the model and runs both gates before it shows you
the result: ``pbip-forge check`` must report no P0 or P1 violations
(:doc:`check-rules`), then ``tmdl-preflight check`` must report no errors
on the built folder. The skills call the two commands on your ``PATH``,
so install both tools first.

Passing both gates proves the files pass both tools' rules. Opening the
model in Power BI Desktop is a step you take yourself; the skills do not
claim it opens.

.. list-table::
   :widths: 20 40 40
   :header-rows: 1

   * - Skill
     - What you give it
     - What you get back
   * - ``/create-sample-model``
     - a business domain, described in your own words
     - a PBIP folder with sample data loaded, plus its hand-off pack:
       ``SPEC.md`` (diagram and data dictionary) and ``manifest.json``
   * - ``/validate-and-expand``
     - an existing model, plus what you want added or checked
     - the same model, rebuilt from the changed spec and checked again

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
  date range (the clinic model's visits run from January 2024 to
  September 2025). The skill invents the rows.
- **A hand-off pack** next to the model: ``handoff/SPEC.md``, with a
  diagram of every table and relationship and a data dictionary, and
  ``handoff/manifest.json``, the same model in machine-readable form. See
  :doc:`handoff-pack`.
- **Both gate results**: no P0 or P1 violations from ``pbip-forge
  check``, and no errors from ``tmdl-preflight check`` on the built
  folder, before it is shown to you.

The repository keeps the clinic model, with the insurance expansion
below already applied, as ``scripts/build_vet_clinic.py``; a model the
skill makes for you will differ in its details.

``/validate-and-expand``
----------------------------

Give it an existing model plus what it is missing, in plain language, for
example "add a pet insurance company entity and measures relating it to
relevant existing entities." What comes back:

- **A new table**, when the request calls for one, at whatever grain
  fits. The veterinary example added an InsuranceProviders table: four
  invented insurers plus a "Self-Pay" entry, each with a plan type and a
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
  or P1 violations, still no errors on the model.
- **The original tables, untouched.** Nothing about the patients, the
  vets, or the services needed to change for the new entity to fit in
  alongside them.

Related topics
--------------

* :doc:`/topic-guides` explains what an ontology means for
  ``/create-sample-model``, and how ``/validate-and-expand`` fits a new
  entity into one that already exists.
