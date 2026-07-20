Build a drill-down matrix
===========================

A matrix visual displays numbers in a grid that you can drill from a broad
level down to a detailed one, such as year down to month down to day, or
category down to product, without building a separate chart for each level.

This continues on the hospital model from
:doc:`build-a-model-from-a-prompt`; open it in Power BI Desktop before
starting.

#. Select the *matrix* visual from the **Visualizations** pane.
#. Drag a hierarchy of columns, such as *Department* then *Physician* from
   the Departments and Physicians tables, to the **rows** field well,
   ordered from broadest to narrowest.
#. Drag a numeric measure, such as *Total Charges*, to the **values** field
   well.
#. Select the visual, then use the drill-down arrows in its header to move
   between levels, or double-click a row to expand it.

.. Screenshot to take: the finished matrix in Power BI Desktop, showing
   department rows expanded to individual physicians, with Total
   Charges as the value column. Save as
   docs/_static/screenshots/hospital-drill-down-matrix.png.

.. image:: /_static/screenshots/hospital-drill-down-matrix.png
   :alt: A drill-down matrix built from the hospital model, with Department rows expanded into individual Physician rows and Total Charges as the value column.
