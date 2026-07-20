Reference
============

pbip-model-forge is a Claude Code skill pair, not a command-line tool: everything
happens through two slash commands typed directly into a Claude Code conversation.
Each command wraps tmdl-preflight validation internally, so every model these
skills produce is already confirmed to open cleanly in Power BI Desktop before you
see it.

.. list-table::
   :widths: 20 40 40
   :header-rows: 1

   * - skill
     - what you give it
     - what you get back
   * - `/create-sample-model`
     - a business domain, described in your own words
     - a working .pbip file, plus a plain-language summary of its tables
   * - `/validate-and-expand`
     - an existing model, plus what you want added or checked
     - the same model, updated and re-validated, still openable in Power BI Desktop

Related topics
--------------

* :doc:`/explanation/ontology-basics` explains what an ontology means for
  `/create-sample-model`.
* :doc:`/explanation/expanding-an-ontology` explains how `/validate-and-expand`
  fits a new entity into one.
