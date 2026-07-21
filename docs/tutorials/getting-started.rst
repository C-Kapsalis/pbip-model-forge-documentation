Getting started
===================

In this tutorial you describe a business domain in a Claude Code
conversation, get back a working Power BI data model, then grow that
model by describing what it is missing. It takes about ten minutes.

You will need Windows and Power BI Desktop, which is free to install,
to open the result and look around.

1. Set up
-------------

Clone the repository, then install ``tmdl-preflight`` from its own
folder; it is the separate validation tool every generated model is
gated on, not something bundled here:

::

    git clone https://github.com/c-kapsalis/pbip-model-forge.git
    git clone https://github.com/c-kapsalis/tmdl-preflight.git
    pip install -e ./tmdl-preflight

Confirm it is on the path:

::

    tmdl-preflight --version

Open a Claude Code session inside the ``pbip-model-forge`` folder; the
rest of this tutorial happens entirely in that conversation.

2. Build your first model
------------------------------

In a Claude Code session in this repository, describe a domain in your
own words:

::

    create a sample data model for a veterinary clinic

The ``create-sample-model`` skill takes it from there: it designs a
star schema for the domain, invents believable sample data, builds the
``.pbip`` file, and confirms it opens cleanly before showing it to you.
For a veterinary clinic, what comes back is a model with:

- **Patients** (8 rows): each animal's name, species, breed, and
  owner.
- **Veterinarians** (4 rows): name and specialty, such as Surgery or
  Dermatology.
- **Services** (8 rows): the clinic's billable service catalog, each
  with a price, from a wellness exam to surgery.
- **Visits**, the fact table behind the scenes: one row per clinic
  visit, tying a patient to a vet and a service on a date, spanning
  early 2024 through late 2025.
- A calendar table, and a measures table with **Total Revenue**,
  **Visit Count**, **Unique Patients**, **Average Visit Cost**, and
  **Total Visit Hours** already defined.

The skill shows the shape of the model as a diagram, tables connected
by their relationships, patients, vets, and services each feeding into
visits, visits feeding into the calendar by date. Alongside it: the
model checked out clean, no errors, no warnings.

3. Add something to it
---------------------------

Describe what is missing the same way, in the same conversation:

::

    add a pet insurance company entity and measures relating it to
    relevant existing entities

The ``validate-and-expand`` skill grows the existing model rather than
starting over. For the veterinary clinic, it added:

- **InsuranceProviders** (5 rows): four real insurers, each with a
  plan type and a coverage rate, plus a "Self-Pay" entry for visits
  with no insurance at all.
- Two new fields on **Visits**: how much was claimed per visit, and
  how much of that was actually covered, each visit now tied to one
  insurer or to self-pay.
- Five new measures: **Total Claims Submitted**, **Total Covered
  Amount**, **Out-of-Pocket Revenue**, **Insurance Reimbursement
  Rate**, and **% Visits Insured**.

The model is checked again after the change, the same way it was
after the first pass, and comes back clean: still no errors, still no
warnings. Nothing about the original model, the patients, the vets,
the services, needed to change for the new entity to fit.

4. Open it in Power BI Desktop
-----------------------------------

The skill tells you where the file landed. Open it in Power BI
Desktop and look around: the diagram view shows every table now
connected, including the new insurance provider, and the sample data
is already loaded, so the measures compute immediately.

.. figure:: /_static/screenshots/vet-clinic-model-open.png
   :alt: The veterinary clinic data model open in Power BI Desktop's model view. Patients, Veterinarians, and Services each connect into the hidden Visits fact table, InsuranceProviders connects to Visits by insurance_provider_id, and the Calendar table connects by date. A Clinic Measures table lists Total Revenue, Visit Count, Average Visit Cost, and the new insurance measures, Total Claims Submitted, Total Covered Amount, and Insurance Reimbursement Rate among them.

   The expanded veterinary clinic model, with InsuranceProviders now
   connected alongside the original tables.

Where to go next
--------------------

- Try the same two prompts on your own domain, or a few more
  examples: :doc:`../how-to/build-a-model-from-a-prompt`
- Add a couple of simple visuals to read the numbers a generated model
  already contains: :doc:`../how-to/build-a-line-and-clustered-column-chart`
  and :doc:`../how-to/build-a-drill-down-matrix`
- What the two skills are and what each one takes and gives back:
  :doc:`../reference/skills`
- What an ontology means here, and how a new one gets fitted into an
  existing model: :doc:`../topic-guides`
