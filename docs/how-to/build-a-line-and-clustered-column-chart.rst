Build a line and clustered column chart
=========================================

A line and clustered column chart shows how two measures move over time:
columns for one value and a line for another, sharing the same date axis.

This builds on the hospital model from :doc:`build-a-model-from-a-prompt`
(``scripts/build_hospital.py``). Open it in Power BI Desktop and go to
the empty ``Page 1`` before you start.

#. Select the *line and clustered column chart* visual in the
   **Visualizations** pane.
#. Drag *Date* from the ``__Calendar`` table to the **X-axis** field well.
#. Drag *Total Billed Revenue*, from the ``Clinic Measures`` table, to the
   **Column y-axis** field well.
#. Drag *Avg Billed per Patient* to the **Line y-axis** field well.
#. Select the visual, open the **Format** pane, and give it a descriptive
   title.

In the sample data every admission date has one encounter and one
patient, so on a daily axis *Avg Billed per Patient* has the same value
as *Total Billed Revenue* on every date.

For a view where the two differ, drag *Month Start* from ``__Calendar`` to
the **X-axis** instead of *Date*. January and March 2024 each had two
patients, so there the line sits below the column: $10,850.00 against
$21,700.00 in January, and $6,950.00 against $13,900.00 in March.

.. figure:: /_static/screenshots/hospital-line-and-column-chart.png
   :alt: A line and clustered column chart titled "Total Billed Revenue and Avg Billed per Patient by Date". Columns for Total Billed Revenue run from early 2024 to May 2025, the tallest about $42K in mid-2024. The Avg Billed per Patient line stays far below the columns, peaking at about $7K.

   The chart in Power BI Desktop. The screenshot was taken with an
   earlier version of the hospital example, in which ``Patients #``
   counted the ``Patients`` table: every point of the line divides by all
   six patients, so the line sits far below the columns (about $7K under
   the $42K column). With the 1.2.1 example the two measures have the
   same value on every date, as described above.
   :doc:`build-a-drill-down-matrix` explains the difference.
