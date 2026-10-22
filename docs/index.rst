pbip-model-forge
=================

pbip-model-forge turns a description of a data model into a Power BI
project you can open and an engineer can build from: a ``.pbip`` folder
with the tables, relationships and measures in place and sample data
already loaded, plus a hand-off pack that documents all of it. These
pages cover version 1.2.1.

The problem this solves
---------------------------

A domain expert who wants a new data model today has one real option:
describe it to a data team and wait. The request joins a ticket queue,
gets interpreted secondhand, and comes back days or weeks later, sometimes
shaped nothing like what was meant.

pbip-model-forge moves that first pass to the person who knows the
domain. You describe the model as a spec, either by writing a JSON file
yourself or by asking one of two Claude Code skills to write it for you
in conversation. ``pbip-forge`` checks the spec and builds the model from
it.

What comes back is not a finished report. The report holds one empty
page, ``Page 1``, and no visuals. What matters is the model underneath
it: the tables, the relationships and the measures, loaded with sample
data so the numbers compute and a stakeholder can see the shape of an
idea before an engineer spends a sprint building it against the
production warehouse. That model, with its hand-off pack (a diagram, a
data dictionary and every measure's DAX), is the spec you hand to your
data team in place of a ticket they have to reverse-engineer.

Two gates
-----------

A model is ready to hand off when it passes two checks, in this order:

#. ``pbip-forge check``, part of this project, judges the spec against 22
   modeling rules: keys that line up, measures counted at the right grain,
   every measure formatted and described. It must report 0 P0 and 0 P1.
#. ``tmdl-preflight``, a separate tool you install alongside it, checks the
   files the build writes. It must report 0 errors.

Since 1.2.1 the report that ``build`` writes passes tmdl-preflight too,
so the second gate runs on the whole built folder. The gates prove the
files pass both tools' rules; opening the model in Power BI Desktop, on
Windows, is a step you take yourself.

Where to start
----------------

- Install the two tools: :doc:`how-to/install`.
- Write a spec yourself and hand the result to engineers:
  :doc:`how-to/build-a-model-from-a-spec`.
- Let Claude Code write the spec in conversation:
  :doc:`tutorials/getting-started`.

The command line
------------------

``pbip-forge`` has eight commands. Three work on a spec:

- ``check`` judges a spec against the 22 modeling rules.
- ``build`` writes the whole PBIP and the hand-off pack from a spec.
- ``rules`` prints the rule catalog.

Five work on a PBIP project that already exists:

- ``table dump`` prints the rows of an encoded "Enter data" table.
- ``report blank`` creates a report with one empty page for a model that
  has none, the report a Fabric Git sync or a deployment pipeline needs.
- ``clone-report`` copies a report folder under a new name and a new
  identity.
- ``f-filters`` fills the ``f-<Category>`` measures and the ``f-All`` gate
  lines of a model built on the selector pattern.
- ``i-filters`` regenerates the ``i-Applied Filters`` caption measure.

:doc:`reference/cli` covers the spec commands and the gates;
:doc:`reference/project-commands` covers the other five.

.. toctree::
   :maxdepth: 1

   tutorials/getting-started
   how-to-guides
   reference
   topic-guides
   contributing
