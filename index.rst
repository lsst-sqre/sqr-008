..
  Content of technical report.

  See http://docs.lsst.codes/en/latest/development/docs/rst_styleguide.html
  for a guide to reStructuredText writing.

  Do not put the title, authors or other metadata in this document;
  those are automatically added.

  Use the following syntax for sections:

  Sections
  ========

  and

  Subsections
  -----------

  and

  Subsubsections
  ^^^^^^^^^^^^^^

  To add images, add the image file (png, svg or jpeg preferred) to the
  _static/ directory. The reST syntax for adding the image is

  .. figure:: /_static/filename.ext
     :name: fig-label
     :target: http://target.link/url

     Caption text.

   Run: ``make html`` and ``open _build/html/index.html`` to preview your work.
   See the README at https://github.com/lsst-sqre/lsst-report-bootstrap or
   this repo's README for more info.

   Feel free to delete this instructional comment.

:tocdepth: 1

.. note::

    Work in progress.


Introduction
============

The Data Quality Assessment Plan (LSE-63) defines four levels of QA summarized
here:

    - **Level 0 QA:** test the DM software during pre-commissioning as well as 
      test software improvements during commissioning and operations, 
      quantifying the software performance against known expected outputs.
    - **Level 1 QA:** data quality in real time during commissioning and 
      operations. Analyzes the data stream in real time, information about 
      observing conditions (sky brightness, transparency, seeing, magnitude 
      limit) as well as characterize subtle deterioration in system performance.
    - **Level 2 QA:** assesses  the quality of data releases 
      (including the co-added image data products) performing quality 
      assessment for astrometric and photometric calibration and derived 
      products, looking for problems with the image processing pipelines and 
      systematic problems with the instrument.
    - **Level 3 QA:** data quality based on science analysis performed by the 
      LSST Science Collaborations and the community. Level 0-2 visualization 
      and data exploration tools will be made available to the community.

During pre-commissioning, the LSST DM Software and its data products are being 
evaluated by processing precursor datasets from surveys like DECam, CFHT and 
HSC. These verification datasets were selected to span a wide range of regimes
and to test Level 1 (Alert Production) and Level 2 (Data Release Production) 
data products.

SQuaRE is concerned about preserving the knowledge developed in this effort
and reuse as much as possible the code in the production Data Quality Assessment 
Framework (DQAF), possibly at all levels of QA.

In practice, that can be achieved by developing a prototype of the DQAF that
would be adopted, maintained and refined by the   *verification
scientists*. At the same time, they have similar needs in terms of data
access, data processing and visualization and would benefit from a common 
infrastructure that facilitates all those aspects. 

The verification datasets use case has specific needs that must be
distinguished from the production system. Since we are processing
multiple datasets, it makes sense to accumulate QA information across multiple runs
in a separate database. However, in production, each run will result in a different
database with associated QA tables (http://ldm-135.readthedocs.org/). Despite those differences,
the end goal of a prototype at this point is to leverage the design of the production QA database.


Single Image Requirements
=========================

LPM-17 is the basis for the present design, it specifies single image requirements
and how to select the datasets to compute them. In this section we summarize the single image
requirements and discuss the selection of the QA datasets based on the outputs available
in the baseline database schema as specified by LSE-163.

TODO: include tables summarizing the single image requirements in LPM-17

TODO: write queries to select datasets for image depth, image quality, astrometry and photometry accuracy based on LPM-17
from the baseline database schema (https://lsst-web.ncsa.illinois.edu/schema/index.php?t=RefObjMatch&sVer=baseline)
and double check with LSE-163


Image depth
-----------

**Sample selection:**  *r* band images in good seeing < 0.7 arcsec (FWHM) on photometric dark nights
(*r*-band sky brightness > 21 :math:`mag/arcsec^2`) close to the zenith (airmass < 1.2) and total
exposure time of 30s.

.. code-block:: sql

    SELECT psFlux, psFluxSigma
    FROM Source s
    INNER JOIN CcdVisit c ON s.ccdVisitId=c.ccdVisit
    INNER JOIN Visit v ON c.visitId=v.visitId
    WHERE s.filterName='r' and c.seeing < 0.7 and s.skySigma > 21 and v.airmass < 1.2 and v.expTime = 30;


**Notes**
    Table 5 (Catalog `Source` table) in LSE-163 does not have `psFluxSigma` and Table 6 (Catalog `ForcedSource` table)
    name this column as `psFluxErr`, however baseline schema has.

    Table 5 (Catalog `Source` table) does not have `filterName` in LSE-163, however baseline schema has.


The overall image depth distribution
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

**Specification:** The distribution of the 5 :math:`\sigma` (SNR=5) detection depth for point sources for all the exposures in
the *r* band will have a median not brighter than D1 mag, and no more than DF1 % of images will have 5 :math:`\sigma` depth
brighter than Z1 mag. D1 and Z1 are expressed in the AB system.


.. _table-depth_distribution:
.. table:: LPM-17 Table 5. Single visit depth distribution in *r* band at SNR=5 for point sources.

    +-------------+-------------+--------------+--------------+
    | Quantity    | Design Spec | Minimum Spec | Stretch Goal |
    +=============+=============+==============+==============+
    | D1 (mag)    |  24.7       |  24.3        |  24.8        |
    +-------------+-------------+--------------+--------------+
    | DF1 (%)     |  10         |  20          |  5           |
    +-------------+-------------+--------------+--------------+
    | Z1 (mag)    |  24.4       |  24          |  24.6        |
    +-------------+-------------+--------------+--------------+

DF1 is the fraction not of all exposures but of those in the selected sample.


The variation of the image depth (throughput) with bandpass
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

**Specification:** The median 5 :math:`\sigma` (SNR=5) detection depth for point sources in a given band will not be brighter than DB1 mag.

.. _table-single_visit_depth:
.. table:: LPM-15 Table 6. Single visit depth (DB1) as function of bandpass

    +-------------+-------------+--------------+--------------+
    | Quantity    | Design Spec | Minimum Spec | Stretch Goal |
    +=============+=============+==============+==============+
    | u DB1 (mag) |  23.9       |  23.4        |  24          |
    +-------------+-------------+--------------+--------------+
    | g DB1 (mag) |  25.0       |  24.6        |  25.1        |
    +-------------+-------------+--------------+--------------+
    | r DB1 (mag) |  24.7       |  24.3        |  24.8        |
    +-------------+-------------+--------------+--------------+
    | i DB1 (mag) |  24.0       |  23.6        |  24.1        |
    +-------------+-------------+--------------+--------------+
    | z DB1 (mag) |  23.3       |  22.9        |  23.4        |
    +-------------+-------------+--------------+--------------+
    | y DB1 (mag) |  22.1       |  21.7        |  22.2        |
    +-------------+-------------+--------------+--------------+

The variation of the image depth over the field of view
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

**Specification:** For an image representative of the median depth (*i.e* with the 5 :math:`\sigma` detection depth
of D1 mag), the depth distribution over individual devices will have no more than DF2 % of the sample brighter by more
than Z2 mag than the median depth.

.. _table-variation-over-fov:
.. table:: LPM-15 Table 7. Image depth variation over the field of view. This apply to all bands.

    +-------------+-------------+--------------+--------------+
    | Quantity    | Design Spec | Minimum Spec | Stretch Goal |
    +=============+=============+==============+==============+
    | DF2 (%)     |  15         |  20          |  10          |
    +-------------+-------------+--------------+--------------+
    | Z2 (mag)    |  0.2        |  0.4         |  0.2         |
    +-------------+-------------+--------------+--------------+





Image Quality
-------------

Astrometry Accurracy
--------------------

Photometry Accurracy
--------------------

The QA database
===============
 
The database is being designed according to some general guidelines: 

- Should store the results of the science requirements and summary information for CCDs and Visits;
- Should be easily extended, i.e additional tests should be included without changing the schema;
- Should be optimized for interactive visualization, i.e the results of the science requirements and summary information
are pre-computed and stored in 'materialized views'  for performance improvements;
- Must be camera agnostic, i.e support DECam, CFHT and HSC images processed by the stack;

The main difference between the verification datasets use case and the production system is that
here the QA results from multiple runs will be accumulated in a single database, including basic provenance information.
Thus, in addition to the science requirements and summary information table we also have process information tables.


.. figure:: _static/sqa.png
   :name: fig-sqa-database
   :target: _static/sqa.png
   :alt: SQuaRE QA database

   QA database model.


The source code is maintained in this repo https://github.com/lsst-sqre/qa-database 

The proposed database has three sets of tables:

**Science Requirements**
    Are based on the single image specification contained in the 
    Science Requirements Document (LPM-17) and associated to each single ccd. 
    Science Requirement descriptions, results, conditions and thresholds are stored in the
    metrics table. 
**Summary Information** 
    They store medians and MADs (Median Absolute Deviations) of interesting 
    properties of each CCD, it enables fast visualization and aggregate 
    quantities computed at the visit level. These properties are 
    computed from a subset of high S/N point sources also stored in the 
    database.
**Process information** 
    They store basic provenance information such as configuration, data 
    repository path, code version and logs of each process. If the full image 
    and source catalogs are required for futher inspection they can be retrieved
    from the data repository using the butler.

It is also being designed to be a *common model* for the different instruments 
supported by the stack. An advantage of that is the comparison of metrics and 
results accross different processes of the same dataset or accross different 
datasets. The mechanism for translating camera-specific metadata to this 
common model is still under discussion.

Sample queries
==============

Process information
-------------------

- Give me all processed datasets, run numbers, date, duration, status and who 
  processed

.. code-block:: sql
     
    SELECT d.name, 
        p.processId, 
        p.start, 
        p.end - p.start as duration, 
        p.status, 
        u.username
    FROM Process p 
    INNER JOIN Dataset d ON p.datasetId = d.datasetId
    INNER JOIN user u ON p.userId = u.userId;

- For run=xxxx, give me the fraction of ccd failures

.. code-block:: sql

    SELECT ptv.nFailure/(ptv.nSuccess + ptv.nFailure) as fraction 
    FROM ProcessToVisit ptv
    INNER JOIN Process p ON ptv.processId=p.processId  
    WHERE processId = 'xxxx';


- For run=xxxx, give me a list of visits with failures

.. code-block:: sql

    SELECT visit 
    FROM Visit v 
    INNER JOIN ProcessToVisit ptv ON v.visitId = ptv.visitId
    INNER JOIN Process p ON ptv.runId = p.runId
    WHERE ptv.nFailure > 0 AND p.processId = 'xxxx';


- Give me the footprint of run xxxx (i.e. corners in sky coordinates of all processed ccds) 

.. code-block:: sql
    
    SELECT c.llra, 
       c.lldec, 
       c.urra, 
       c.urdec 
    FROM Ccd c 
    INNER JOIN Visit v ON c.visitId = v.visitId
    INNER JOIN ProcessToVisit ptv ON v.visitId = ptv.visitId
    INNER JOIN Process p ON ptv.processId = p.processId 
    WHERE p.processId = 'xxxx';
 
- Give me the configuration and version of the stack used to process visit yyyy
  in run xxxx

TODO: include ``stackVersion`` in Process table

.. code-block:: sql

    SELECT p.config,
        p.stackVersion
    FROM Visit v, 
    INNER JOIN ProcessToVisit ptv ON v.visitId = ptv.visitId
    INNER JOIN Process p ON ptv.processId = p.processId 
    WHERE v.visit = 'yyyy'
    AND p.processId = 'xxxx';
 
   
Summary Information
-------------------

- Give me filter, exposure time, zenith distance, air mass, hour angle, fwhm, ellipticity, sky background and the scatter in ra and decl of all ccds in visit yyyy  


.. code-block:: sql

    SELECT v.filter, 
        v.exposureTime, 
        v.zenithDistance, 
        v.airMass, 
        v.hourAngle, 
        c.medianFwhm, 
        1.0-c.medianMinorAxis/c.medianMajorAxis as ellipticity,
        c.medianSkyBg,
        c.medianScatterRa,
        c.medianScatterDecl
    FROM Visit v,
        Ccd c
    WHERE v.visitId = c.visitId 
    AND v.visit = 'yyyy';


- Give me summary information for all visits processed by run xxxx (use ``scisql_median()``  to aggregate values per visit)

.. code-block:: sql

    SELECT v.visit,
       v.filter, 
       v.exposureTime, 
       v.zenithDistance, 
       v.airMass, 
       v.hourAngle, 
       scisql_median(c.medianFwhm) as fwhm, 
       scisql_median(1.0-c.medianMinorAxis/c.medianMajorAxis) as ellipticity,
       scisql_median(c.medianSkyBg) as skyBg,
       scisql_median(c.medianScatterRa) as scatterRa,
       scisql_median(c.medianScatterDecl) as scatterDecl
    FROM Ccd c 
    INNER JOIN Visit v ON c.visitId = v.visitId
    INNER JOIN ProcessToVisit ptv ON v.visitId = ptv.visitId
    INNER JOIN Process p ON ptv.processId = p.processId where ProcessId = 'xxxx'
    GROUP BY v.visit,
         v.filter,
         v.exposureTime,
         v.zenithDistanced,
         v.airMass,
         v.hourAngle
    ORDER BY v.visit;

- Give me the process ccd logs of failed ccds in visit yyyy

.. code-block:: sql

    SELECT c.log
    FROM Visit v, 
        Ccd c
    WHERE v.visitId = c.visitId 
    AND v.visit = 'yyyy'
    AND c.status = 1;


- Give me the source catalog and image FITS files for ccd c, visit yyyy procesed
  by run xxxx 

  Can't be done in SQL, but an API can return the ``outputDir`` and then one can
  use the butler to get files giving the ccd and visit. 

- Give me median scatter in RA and Dec for all visits in all runs that processed
  dataset=zzz, the version of the stack, the configuration file used, 
  from date=yyyy-mm-dd 

.. code-block:: sql

    SELECT p.processId as run,
        v.visit,
        scisql_median(c.medianScatterRa) as ra_scatter,
        scisql_median(c.medianScatterDecl) as dec_scatter,
        p.stackVersion,
        p.config
    FROM Ccd c 
        INNER JOIN Visit ON c.visitId = v.visitId
        INNER JOIN ProcessToVisit ptv ON v.visitId = ptv.visitId
        INNER JOIN Process p ON ptv.processId = p.processId 
        INNER JOIN dataset d ON p.datasetId = d.datasetId
    WHERE d.name = 'zzzz'
    AND p.start > 'yyy-mm-dd'
    GROUP BY v.visit
    ORDER BY p.processId;


- Recover DECam image quality history (e.g. fwhm and its scatter) from date 
  yyyy-mm-dd to yyyy-mm-dd looking at all runs that processed decam dataset

.. code-block:: sql

    SELECT p.processId as run,
       d.name as dataset,
       v.visit,
       scisql_median(c.medianFwhm) as fwhm,
       scisql_median(c.madFwhm) as scatter
    FROM Ccd c 
    INNER JOIN Visit ON c.visit_id = v.visit_id
    INNER JOIN ProcessToVisit ptv ON v.visitId = rv.visitId
    INNER JOIN Process p ON ptv.processId = p.processId 
    INNER JOIN Dataset d ON p.datasetId = d.datasetId
    WHERE d.camera = 'decam'
    AND p.start > 'yyyy-mm-dd'
    AND p.end < 'yyyy-mm-dd'
    GROUP BY v.visit
    ORDER BY p.processId;


QA Metrics
----------

- Give me all metrics descriptions, conditions and thresholds available
- Give me all metrics where at least one ccd failed in run xxx
- Give me the fraction of visits in run xxxx that passed metric mmmm
- Give me the value and sigma of the metric mmmm for all ccds in visit yyyy


References
----------

  - LSE-63 Data Quality Assurrance Plan
  - LPM-17 Science Requirements Document
  - LDM-135: Database Design 
  - LSST Database Schema, baseline version (https://lsst-web.ncsa.illinois.edu/schema/index.php?sVer=baseline)
  - pipeQA
  - HSC Database schema v1.0 
  - DES Quick Reduce and DES operations database

