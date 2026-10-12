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
   :alt: A line and clustered column chart titled "Total Billed Revenue and Avg Billed per Patient by Date." One column per admission date for Total Billed Revenue, from about $3K to $42K across 2024 and into mid-2025, with the tallest column in late June 2024. The Avg Billed per Patient line runs through the top of every column, because each admission date in the sample has one encounter and one patient.

   Total Billed Revenue and Avg Billed per Patient, plotted by admission
   date across the hospital's encounter history.

Each admission date in the sample data has a single patient, so on a daily
axis the average equals the revenue. For a view where the two differ, drag
*Month Start* from the Calendar table to the **X-axis** instead of *Date*:
in January and March 2024 two patients were billed, so the line sits below
the column ($10,850.00 against $21,700.00, and $6,950.00 against
$13,900.00).
