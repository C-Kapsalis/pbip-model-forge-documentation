Contributing
============

This page is for contributors extending pbip-model-forge itself or its two
Claude Code skills: people comfortable editing TMDL directly and changing
a ``SKILL.md`` file. For development setup, running the test suite and
pull request conventions, see `CONTRIBUTING.md
<https://github.com/C-Kapsalis/pbip-model-forge/blob/v1.2.1/CONTRIBUTING.md>`_
in the repository root; this page covers the concepts behind the code.

The pipeline
------------

Every model pbip-model-forge produces comes from the same four-step
pipeline, whether a skill or a person wrote the spec.

#. A spec dict describes the model: its tables, columns, sample rows,
   measures and relationships (:doc:`reference/spec-format`). This dict is
   the single source of truth.
#. ``pbip-forge check`` judges the spec against the modeling rules in
   :doc:`reference/check-rules`. No P0 or P1 violations means the model is
   correct enough to hand off.
#. ``build_pbip(spec, out_dir)`` renders every TMDL and JSON file the model
   needs from that dict: the compressed entered-data payload that loads
   the sample rows, a complete PBIR report with one empty page, and the
   hand-off pack. Every id is derived from the object's name, so the same
   spec always yields the same files.
#. ``tmdl-preflight check`` gates the result: the whole output folder,
   model and report, must report 0 errors.

TMDL is always regenerated from the spec dict, never hand-edited, and
neither skill hand-authors TMDL or base64.

How the skills use it
---------------------

Each skill lives in ``.claude/skills/<name>/SKILL.md``, a Markdown file
with a YAML frontmatter block (``name``, ``description``). Claude Code
matches a user's request against that description to decide which skill
to invoke.

* ``create-sample-model`` invents a spec dict from a domain description,
  then runs the pipeline above: check, build, then gate.
* ``validate-and-expand`` edits an existing spec dict to add whatever was
  requested (a table, a column, a measure, a relationship), then runs the
  same check, build and gate sequence. It never patches generated TMDL by
  hand; every expansion goes through the spec dict first.

Both skills share the same invariant: the spec dict is the source of truth,
``pbip-forge check`` decides whether the model is correct, and
``tmdl-preflight check`` decides whether the files pass its rules.

Entered-data encoding
---------------------

Sample rows are not read from an external data source. They are loaded
into the model through a compressed, base64-encoded "entered data"
partition, the same construct Power BI Desktop's own **Enter Data**
feature produces. Inline ``#table(...)`` sources trip tmdl-preflight
rule M008 (the composite-model error); the entered-data partition does
not, and it needs no file or connection outside the PBIP folder.

``src/pbip_model_forge/enter_data.py`` is the encoder. It has to reproduce
Power BI Desktop's own encoding byte for byte: raw DEFLATE, compact JSON
separators, every cell a string. ``tests/test_enter_data.py`` holds three
golden base64 strings emitted by Power BI Desktop, and
``test_encoder_matches_powerbi_golden`` and ``test_roundtrip`` check the
encoder against them. Any encoder change must keep both passing.

Extending
---------

The most common ways to extend the builder each touch a different layer of
the pipeline above.

The five commands that work on an existing project live in their own
modules: ``table dump`` uses the entered-data codec in ``enter_data.py``,
``report blank`` and ``clone-report`` are in ``report_scaffold.py``, and the
two selector-pattern repairers are ``f_filters.py`` and ``i_filters.py``.
:doc:`reference/project-commands` documents them, with their options and
their console output.

Add a column type
~~~~~~~~~~~~~~~~~

Map the new keyword to a ``dataType`` and a ``TransformColumnTypes``
target in ``TYPE_MAP`` (``src/pbip_model_forge/enter_data.py``). The
``dataType`` has to be one tmdl-preflight already recognizes. Add a
coercion branch to ``stringify`` if the value needs special formatting,
and a golden or round-trip case to ``tests/test_enter_data.py``.

Add a new generator
~~~~~~~~~~~~~~~~~~~

Add a ``render_*`` function in ``build_model.py`` and wire it into
``build_pbip``. Keep two invariants whatever the generator produces:
every table gets a ``ref table`` line in ``model.tmdl``, and every table
gets at least one partition. Rebuild the template and run both gates on
it; the template must pass ``tmdl-preflight`` with 0 errors:

::

    python scripts/build_template.py
    pbip-forge check scripts/build_template.py --profile handoff
    tmdl-preflight check template

At 1.2.1 these print:

::

    Built template PBIP: /path/to/pbip-model-forge/template/Model.pbip
    pbip-forge check (handoff): 0 P0, 0 P1, 0 P2
    tmdl-preflight: 0 error(s), 0 warning(s), 1 info(s)

The last line is the end of tmdl-preflight's output; the one info is the
advisory R013 on the calendar's column names.

Add a check rule
~~~~~~~~~~~~~~~~

Add a ``Rule`` to the catalog in ``src/pbip_model_forge/check.py`` with
the next free ``FM`` number, a severity for each profile, a rationale and
a fix, plus the function that detects it. Add a test to
``tests/test_check.py`` that trips it from ``good_spec()`` in
``tests/specs.py``, and add the rule to :doc:`reference/check-rules`.

Add or change a skill
~~~~~~~~~~~~~~~~~~~~~

Edit the relevant ``.claude/skills/<name>/SKILL.md``. Keep the golden rule
visible in the skill's own instructions: never hand-author TMDL or base64,
always go through ``build_pbip``, and always gate the result on
``pbip-forge check`` and then ``tmdl-preflight``.
