Expanding an ontology
=====================

Once a model exists, new entities tend to turn up that were not part of the
original description - a returns process a bakery model did not originally cover,
or a referring provider a clinic model left out.

`/validate-and-expand` is the skill that handles this. Tell it what is missing, in
plain language, and it works out where the new entity belongs: which existing
entities it relates to, which relationship type connects them, and whether it
needs a new table, a new column, or just a new measure on something that already
exists.

Every change runs back through the same validation the original model passed, so
the result stays a model you can open in Power BI Desktop, not a set of
disconnected additions.
