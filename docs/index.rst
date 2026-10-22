pbip-model-forge
=================

Describe a business domain in plain language, and pbip-model-forge
turns it into a real, working Power BI data model: a ``.pbip`` file
with sample data already loaded, ready to open directly in Power BI
Desktop. There is no modeling view to learn and no data model syntax
to write.

The problem this solves
---------------------------

A domain expert who wants a new data model today has one real option:
describe it to a data team and wait. The request joins a ticket
queue, gets interpreted secondhand, and comes back days or weeks
later, sometimes shaped nothing like what was actually meant.

pbip-model-forge collapses that first pass into a conversation.
Describe the domain, and two Claude Code skills design a model,
populate it with believable sample data, and confirm it opens cleanly,
all before a data engineer ever picks up the ticket.

What comes back is not a finished report; the report stays
intentionally blank. What matters is the model underneath it: the
tables, the relationships, the measures, populated with sample data
so the numbers actually compute and a stakeholder can see the real
shape of an idea before an engineer spends a sprint building it
against the production warehouse. That model, plus a plain-language
summary of what it contains, is the spec you hand to your data team,
in place of a ticket they have to reverse-engineer.

Every model these skills produce passes two checks before you ever see
it. ``pbip-forge check``, part of this project, confirms the model
follows modeling best practice: keys that line up, measures counted at
the right grain, every measure formatted and described.
``tmdl-preflight``, a separate validation tool installed once alongside
this project, confirms the files are structurally sound enough for
Power BI Desktop to open. Both run automatically inside the
conversation; you never run them yourself or read their output
directly. One detail of the second check is a property of the tool
rather than of your model: the blank report folder the builder writes is
a two-file stub, and two of tmdl-preflight's report rules read it as an
error. :doc:`reference/cli` gives both gate lines and the exact
commands.

The command line
-----------------

The skills are the way in, but ``pbip-forge`` is a command-line tool
with eight commands, and five of them work on a project that already
exists on disk:

- ``check``, ``build`` and ``rules`` work on a spec: they judge a model
  against the 22 modeling rules, write the whole PBIP from a spec, and
  print the rule catalog.
- ``table dump`` prints the rows of an encoded "Enter data" table.
- ``report blank`` creates a blank report bound to a model that already
  exists, the report a Fabric Git sync or a deployment pipeline needs.
- ``clone-report`` copies a finished report folder under a new name and
  a new identity.
- ``f-filters`` fills the ``f-<Category>`` gate measures and the
  ``f-All`` gate lines of a model built on the selector pattern.
- ``i-filters`` regenerates the ``i-Applied Filters`` caption measure.

:doc:`reference/cli` is the reference: what each command does, the flags
that matter, and what the tool prints.

.. toctree::
   :maxdepth: 1

   tutorials/getting-started
   how-to-guides
   reference
   topic-guides
   contributing
