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

*Avg Billed per Patient* divides revenue by the number of distinct
patients seen in each cell:

::

    Patients # = DISTINCTCOUNT ( Encounters[patient_id] )
    Avg Billed per Patient = DIVIDE ( [Total Billed Revenue], [Patients #] )

Cardiology billed $109,800.00 across three patients, so its average is
$36,600.00. Orthopedics comes to $29,250.00 (two patients), Internal
Medicine $4,000.00, Emergency $3,933.33, and Pediatrics $3,700.00 (three
patients each). The grand total is $203,200.00 over all six patients,
$33,866.67 per patient.

.. note::

   Count an entity from the fact table, not from its own dimension.
   Earlier versions of the hospital model counted
   ``DISTINCTCOUNT ( Patients[patient_id] )``. Departments filter the
   Encounters table, not the Patients table, so every department divided
   by all six patients and Cardiology showed $18,300.00 instead of
   $36,600.00. Counting ``Encounters[patient_id]`` lets every filter
   reach the count. ``pbip-forge check`` reports this pattern as rule
   :ref:`FM015 <fm015>`, so newly generated models cannot ship with it.

.. figure:: /_static/screenshots/hospital-drill-down-matrix.png
   :alt: A matrix with department_name rows, each expanded to show the insurance companies billed within it, and two value columns, Total Billed Revenue and Avg Billed per Patient. Cardiology totals $109,800.00 revenue ($91,300.00 of it Medicare), Orthopedics $58,500.00, Internal Medicine $12,000.00, Pediatrics $11,100.00, and Emergency $11,800.00. The grand total row reads $203,200.00 in revenue and $33,866.67 average billed per patient.

   Revenue by department, drilled down into the insurance company
   billed within each one.
