Ontology basics
================

An ontology, in the context of pbip-model-forge, is the set of entities a
business domain contains and how those entities relate to each other. A
bakery's ontology might include customers, orders, and products; a
clinic's might include patients, appointments, and providers.

When you run ``/create-sample-model``, the skill turns your description of
a domain into exactly that: a set of entities (tables), the properties that
describe each one (columns), and the connections between them
(relationships). It writes them as a spec (:doc:`/reference/spec-format`),
and ``pbip-forge build`` turns the spec into a Power BI semantic model.

You do not need to plan this structure yourself. Describe the business
domain in your own words, and the skill proposes a starting ontology for
it. If you would rather plan it, write the spec yourself:
:doc:`/how-to/build-a-model-from-a-spec`.
