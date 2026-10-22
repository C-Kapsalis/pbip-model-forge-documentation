Expanding an ontology
=====================

Once a model exists, new entities tend to turn up that were not part of the
original description: a returns process a bakery model did not cover, or a
referring provider a clinic model left out.

``/validate-and-expand`` is the skill that handles this. Tell it what is
missing, in plain language, and it works out where the new entity belongs:
which existing entities it relates to, which relationship connects them,
and whether it needs a new table, a new column, or only a new measure on
something that already exists.

The skill makes the change in the spec, never in the generated files, and
rebuilds the model from it. The result goes through the same two gates the
original model passed, ``pbip-forge check`` and ``tmdl-preflight``, so the
new entity arrives connected to the rest of the model rather than as a
set of disconnected additions.
