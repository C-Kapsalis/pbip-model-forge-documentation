Reference
============

pbip-model-forge is a Claude Code skill pair, not a command-line tool: everything
happens through two slash commands typed directly into a Claude Code conversation.
Each command wraps tmdl-preflight validation internally, so every model these
skills produce is already confirmed to open cleanly in Power BI Desktop before you
see it.

.. list-table::
   :widths: 20 40 40
   :header-rows: 1

   * - skill
     - what you give it
     - what you get back
   * - `/create-sample-model`
     - a business domain, described in your own words
     - a working .pbip file, plus a plain-language summary of its tables
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
- **A handful of dimension tables**, each with a key column and a few
  descriptive attributes. For the veterinary clinic: Patients (each
  animal's name, species, breed, and owner), Veterinarians (name and
  specialty), and Services (a billable service catalog with prices).
- **A calendar table** and **a measures table** with several DAX measures
  already defined and formatted, for the clinic: Total Revenue, Visit
  Count, Unique Patients, Average Visit Cost, and Total Visit Hours.
- **Believable sample rows already loaded** in every table, enough that
  the measures compute to real numbers immediately, spanning a plausible
  date range (the clinic model's visits ran from early 2024 through late
  2025).
- **A diagram** of the whole thing: every table, its columns and types,
  and how the tables connect, so you can see the shape of the model
  without opening it.
- **Confirmation the model is clean**: zero errors, zero warnings, before
  it is ever shown to you.

``/validate-and-expand``
----------------------------

Give it an existing model plus what it is missing, in plain language, for
example "add a pet insurance company entity and measures relating it to
relevant existing entities." What comes back:

- **A new table**, when the request calls for one, at whatever grain
  fits, the veterinary example added an InsuranceProviders table: four
  real insurers plus a "Self-Pay" entry, each with a plan type and a
  coverage rate.
- **New fields on the existing fact table**, wired to the new entity: the
  clinic's Visits table gained a foreign key to the insurer, plus how
  much was claimed and how much was actually covered, per visit.
- **New measures**, reachable the same way the original ones are, by
  patient, by vet, by service, or by date. The insurance expansion added
  five: Total Claims Submitted, Total Covered Amount, Out-of-Pocket
  Revenue, Insurance Reimbursement Rate, and % Visits Insured.
- **A refreshed diagram and a re-validation**, the same two things the
  first build produced, confirming the change fits cleanly: still zero
  errors, still zero warnings.
- **The original tables, untouched.** Nothing about the patients, the
  vets, or the services needed to change for the new entity to fit in
  alongside them.

Related topics
--------------

* :doc:`/topic-guides` explains what an ontology means for
  `/create-sample-model`, and how `/validate-and-expand` fits a new
  entity into one that already exists.
