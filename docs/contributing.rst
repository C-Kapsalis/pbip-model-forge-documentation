Contributing
============

This page is for contributors extending pbip-model-forge itself or its two
Claude Code skills - people comfortable editing TMDL directly and modifying a
`SKILL.md` file. For development setup, running the test suite, and pull
request conventions, see `CONTRIBUTING.md
<https://github.com/C-Kapsalis/pbip-model-forge/blob/main/CONTRIBUTING.md>`_ in
the repository root; this page covers the concepts behind the code instead.

The pipeline
------------

Every model pbip-model-forge produces comes from the same three-step
pipeline, regardless of which skill triggered it.

#. A spec dict describes the model: its tables, columns, sample rows,
   measures, and relationships. This dict is the single source of truth.
#. `build_pbip(spec, out_dir)` renders every TMDL and JSON file the model
   needs from that dict, including the compressed entered-data blob that
   physically loads the sample rows.
#. `tmdl-preflight check` gates the result. Zero errors means the model is
   structurally confirmed to open in Power BI Desktop.

TMDL is always regenerated from the spec dict, never hand-edited, and
neither skill hand-authors TMDL or base64 directly.

How the skills use it
---------------------

Each skill lives in `.claude/skills/<name>/SKILL.md`, a Markdown file with a
YAML frontmatter block (`name`, `description`). Claude Code matches a user's
request against that description to decide which skill to invoke.

* `create-sample-model` invents a spec dict from a domain description, then
  runs the pipeline above: build, then gate.
* `validate-and-expand` edits an existing spec dict to add whatever was
  requested - a table, a column, a measure, a relationship - then re-runs the
  same build-and-gate sequence. It never patches already-generated TMDL by
  hand; every expansion goes back through the spec dict first.

Both skills share the same invariant: the spec dict is the source of truth,
and `tmdl-preflight check` is the only thing that gets to call a model valid.

Entered-data encoding
---------------------

Sample rows are not read from an external data source. They are physically
loaded into the model through a compressed, base64-encoded "entered data"
partition - the same construct Power BI Desktop's own **Enter Data** feature
produces, reused here so a generated model never trips a composite-model
error and never depends on a file or connection existing outside the PBIP
folder itself.

`src/pbip_model_forge/enter_data.py` is the encoder. It has to stay
byte-for-byte identical to Power BI Desktop's own output: a single change to
the raw-DEFLATE window bits or the JSON separators is enough to make the
result unopenable. Three golden-value regression tests in
`tests/test_enter_data.py` lock the encoder's output against base64 strings
taken from a model confirmed to open, and they must keep passing for any
encoder change.

Extending
---------

The three most common ways to extend the builder each touch a different
layer of the pipeline above.

Add a column type
~~~~~~~~~~~~~~~~~

Map the new keyword to a `dataType` and a `TransformColumnTypes` target in
`TYPE_MAP` (`src/pbip_model_forge/enter_data.py`). The `dataType` has to be
one tmdl-preflight already recognizes. Add a coercion branch to `stringify`
if the value needs special formatting, and a golden or round-trip case to
`tests/test_enter_data.py`.

Add a new generator
~~~~~~~~~~~~~~~~~~~

Add a `render_*` function in `build_model.py` and wire it into `build_pbip`.
Keep two invariants regardless of what the generator produces: every table
gets a `ref table` line in `model.tmdl`, and every table gets at least one
partition. Rebuild the template and run `tmdl-preflight check template`
afterward.

Add or change a skill
~~~~~~~~~~~~~~~~~~~~~

Edit the relevant `.claude/skills/<name>/SKILL.md`. Keep the golden rule
visible in the skill's own instructions: never hand-author TMDL or base64,
always go through `build_pbip`, and always gate the result on
`tmdl-preflight`.
