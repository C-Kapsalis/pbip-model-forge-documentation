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

Every model these skills produce is checked against ``tmdl-preflight``,
a validation tool that confirms a model is structurally sound before
you ever see it. It needs to be installed once, alongside this
project; from there, checking a model is automatic and happens inside
the conversation. You never run it yourself or read its output
directly.

.. toctree::
   :maxdepth: 1

   tutorials/getting-started
   how-to-guides
   reference/skills
   topic-guides
   contributing
