Build a line and clustered column chart
=========================================

A line and clustered column chart shows how a measure moves over time: columns
for one value, such as a count, and a line for another, such as a running
average, sharing the same date axis. It is a good first chart for spotting
trends without building two separate visuals.

This continues on the hospital model from
:doc:`build-a-model-from-a-prompt`; open it in Power BI Desktop before
starting.

#. On the report canvas, select the *line and clustered column chart* visual
   from the **Visualizations** pane.
#. Drag a date column, such as *Admit Date* from the Admissions table, to the
   **X-axis** field well.
#. Drag a numeric measure, such as *Total Charges*, to the **column values**
   field well.
#. Drag a second measure, such as *Average Charge Amount*, to the **line
   values** field well.
#. Select the visual, open the **Format** pane, and give it a descriptive
   title.

.. Screenshot to take: the finished chart in Power BI Desktop, with
   admissions charges as columns and the average charge as a line over
   time. Save as
   docs/_static/screenshots/hospital-line-and-column-chart.png.

.. image:: /_static/screenshots/hospital-line-and-column-chart.png
   :alt: A line and clustered column chart built from the hospital model, showing Total Charges as columns and Average Charge Amount as a line, both plotted by Admit Date.
