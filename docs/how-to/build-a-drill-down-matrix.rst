Build a drill-down matrix
===========================

A matrix visual displays numbers in a grid that you can drill from a broad
level down to a detailed one, such as year down to month down to day, or
category down to product, without building a separate chart for each level.

This continues on the hospital model from
:doc:`build-a-model-from-a-prompt`; open it in Power BI Desktop before
starting.

#. Select the *matrix* visual from the **Visualizations** pane.
#. Drag *department_name*, from the Departments table, to the **rows**
   field well.
#. Drag *company_name*, from the InsuranceCompanies table, to the **rows**
   field well too, beneath *department_name*, so it becomes the
   drill-down level.
#. Drag *Total Billed Revenue* and *Avg Billed per Patient*, both from
   Clinic Measures, to the **values** field well.
#. Select the visual, then use the drill-down arrows in its header to move
   between levels, or select the expand icon next to a department to show
   its insurers.

.. figure:: /_static/screenshots/hospital-drill-down-matrix.png
   :alt: A matrix with department_name rows, each expanded to show the insurance companies billed within it, and two value columns, Total Billed Revenue and Avg Billed per Patient. Cardiology totals $109,800.00 revenue ($91,300.00 of it Medicare), Orthopedics $58,500.00, Internal Medicine $12,000.00, Pediatrics $11,100.00, and Emergency $11,800.00. The grand total row reads $203,200.00 in revenue and $33,866.67 average billed per patient.

   Revenue by department, drilled down into the insurance company
   billed within each one.
