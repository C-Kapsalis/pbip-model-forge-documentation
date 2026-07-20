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
#. Drag *Date* from the Calendar table to the **X-axis** field well.
#. Drag *Total Billed Revenue*, from the Clinic Measures table, to the
   **column values** field well.
#. Drag *Avg Billed per Patient* to the **line values** field well.
#. Select the visual, open the **Format** pane, and give it a descriptive
   title.

.. figure:: /_static/screenshots/hospital-line-and-column-chart.png
   :alt: A line and clustered column chart titled "Total Billed Revenue and Avg Billed per Patient by Date." Monthly columns for Total Billed Revenue range from about $3K to $42K across 2024 and into mid-2025, with a July 2024 peak of $42K; a line for Avg Billed per Patient tracks a much smaller range, mostly under $10K, moving independently of the columns.

   Total Billed Revenue and Avg Billed per Patient, plotted by month
   across the hospital's encounter history.
