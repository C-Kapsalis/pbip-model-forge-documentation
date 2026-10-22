Getting started
===================

In this tutorial you describe a business domain in a Claude Code
conversation, get back a Power BI data model with sample data loaded,
then grow that model by describing what it is missing. Along the way you
run the two gates yourself on the repository's copy of the same model,
so you see exactly what they print.

You will need Claude Code, and Windows with Power BI Desktop (free to
install) for the last step, to open the result and look around.

1. Set up
-------------

Install ``pbip-forge`` and ``tmdl-preflight`` 1.2.1 and clone the tool's
repository at the same tag, as :doc:`../how-to/install` shows. In short,
with uv:

::

    uv tool install git+https://github.com/C-Kapsalis/pbip-model-forge@v1.2.1
    uv tool install git+https://github.com/C-Kapsalis/tmdl-preflight@v1.2.1
    git clone --branch v1.2.1 https://github.com/C-Kapsalis/pbip-model-forge.git
    cd pbip-model-forge

These commands are the same in PowerShell and in bash. Confirm both tools
are on the path:

::

    pbip-forge --version
    tmdl-preflight --version

::

    pbip-forge 1.2.1
    tmdl-preflight 1.2.1

Open a Claude Code session inside the ``pbip-model-forge`` folder. The
two skills live there, under ``.claude/skills``, and they call the two
commands you just installed.

2. Build your first model
------------------------------

In the Claude Code session, describe a domain in your own words:

::

    create a sample data model for a veterinary clinic

The ``create-sample-model`` skill takes it from there: it designs a star
schema for the domain, invents sample data, writes the spec, checks it
against the modeling rules, builds the ``.pbip`` folder and runs
tmdl-preflight on it before showing it to you. Claude writes a new model
each time, so yours will differ in its details. For the veterinary
clinic, the model that came back had:

- **Patients** (8 rows): each animal's name, species, breed, and owner.
- **Veterinarians** (4 rows): name and specialty, such as Surgery or
  Dermatology.
- **Services** (8 rows): the clinic's billable service catalog, each with
  a price, from a wellness exam to surgery.
- **Visits**, the hidden fact table: one row per clinic visit, tying a
  patient to a vet and a service on a date.
- A calendar table, and a measures table, ``Clinic Measures``, with
  **Total Revenue**, **Visits #**, **Patients #**, **Avg Cost per
  Visit**, and **Total Visit Hours**, each formatted and described.

The skill shows the shape of the model as a diagram taken from the
generated ``handoff/SPEC.md``, and reports both gate results: no P0 or P1
violations from ``pbip-forge check``, and no errors from
``tmdl-preflight``.

The repository keeps this clinic model, with the expansion from step 3
already applied, as ``scripts/build_vet_clinic.py``. Run the two gates on
it yourself, in a terminal in the ``pbip-model-forge`` folder:

::

    pbip-forge check scripts/build_vet_clinic.py --profile handoff
    pbip-forge build scripts/build_vet_clinic.py out/VetClinic --check --profile handoff
    tmdl-preflight check out/VetClinic

The first command prints:

::

    pbip-forge check (handoff): 0 P0, 0 P1, 0 P2

The second:

::

    Built PBIP: out/VetClinic/VetClinic.pbip

The third prints a summary for each of its relationship and calendar
rules and ends with the line that matters:

::

    tmdl-preflight: 0 error(s), 0 warning(s), 1 info(s)

The one info, R013, is advisory: the generated calendar names its
columns ``Calendar Year``, ``Calendar Month`` and so on, not ``Year`` and
``Month``. :doc:`../reference/cli` explains both gates.

3. Add something to it
---------------------------

Describe what is missing the same way, in the same conversation:

::

    add a pet insurance company entity and measures relating it to
    relevant existing entities

The ``validate-and-expand`` skill changes the existing spec rather than
starting over, then rebuilds and runs both gates again. For the
veterinary clinic, it added:

- **InsuranceProviders** (5 rows): four invented insurers, each with a
  plan type and a coverage rate, plus a "Self-Pay" entry for visits with
  no insurance at all.
- New fields on **Visits**: the insurer, how much was claimed per visit,
  and how much of that was covered.
- Six new measures: **Total Claims Submitted**, **Total Covered Amount**,
  **Total Out-of-Pocket Revenue**, **Insurance Reimbursement %**,
  **Insured Visits #**, and **Insured Visits %**.

The model came back with both gates passing again. Patients, vets and
services did not change for the new entity to fit.

4. Open it in Power BI Desktop
-----------------------------------

The skill tells you where the folder landed. On Windows, open the
``.pbip`` file in Power BI Desktop. The sample data is loaded, so the
measures compute as soon as you use them. The report has one empty page,
``Page 1``.

.. figure:: /_static/screenshots/vet-clinic-model-open.png
   :alt: The veterinary clinic model open in Power BI Desktop's model view. Patients, Veterinarians, Services and InsuranceProviders each connect to the hidden Visits fact table, and __Calendar connects to Visits by two date relationships, one of them dotted. A Clinic Measures table lists % Visits Insured, Average Visit Cost, Insurance Reimbursement Rate, Insured Visit Count, Out-of-Pocket Revenue, Total Claims Submitted, Total Covered Amount, Total Revenue and Total Visit Hours.

   An expanded veterinary clinic model in Power BI Desktop's model view.
   The screenshot was taken with an earlier version, so some measure
   names differ from the 1.2.1 example: it shows ``Average Visit Cost``,
   ``% Visits Insured``, ``Insured Visit Count``, ``Insurance
   Reimbursement Rate`` and ``Out-of-Pocket Revenue``.

Where to go next
--------------------

- Write a spec yourself, without Claude Code, and hand the result to
  engineers: :doc:`../how-to/build-a-model-from-a-spec`
- Try the same two prompts on your own domain:
  :doc:`../how-to/build-a-model-from-a-prompt`
- Add a couple of visuals to read the numbers a model already contains:
  :doc:`../how-to/build-a-line-and-clustered-column-chart` and
  :doc:`../how-to/build-a-drill-down-matrix`
- Every key a spec can have: :doc:`../reference/spec-format`
- Every rule a model is checked against: :doc:`../reference/check-rules`
- What an ontology means here, and how a new one gets fitted into an
  existing model: :doc:`../topic-guides`
