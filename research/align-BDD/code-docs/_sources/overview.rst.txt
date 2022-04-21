Implementation: overview
------------------------

These pages document the implementation of the numerical experiments for "Align-BDD" working paper by Bochkarev and Smith. These notes along with the accompanying source code and data are assumed to provide enough information to:

(1) inspect the raw data,
(2) reproduce and adjust/inspect as nececssary the figures from the paper (given the experiment log files), and
(3) completely reproduce the paper results from scratch, including instance generation, with minimal efforts.

All technical questions can be addressed to `Alexey Bochkarev <https://www.bochkarev.io/contact>`_.

Computational infrastructure
----------------------------
To run the experiments we used machines running GNU/Linux system (laptops and remote computational resources provided by `Clemson Palmetto cluster <https://www.palmetto.clemson.edu/palmetto/about/>`_), running the following software: 

* experiments implementation: python3 (:download:`python-packages.txt`),
* figures: R (:download:`R-libraries.txt`),
* parallel computation: `GNU parallel <https://www.gnu.org/software/parallel/>`_,
* experiments pipeline: GNU make and PBS (for job scheduling on the cluster).
* (we also used a separate `tool <https://github.com/alex-bochkarev/tgs-curl>`_ to receive the results promptly via `Telegram <https://telegram.org>`_, but this can be safely removed)
* numerical experiments require `Gurobi <https://www.gurobi.com>`_ solver.

Reproducibility
---------------
Assuming the necessary software is present, one can rebuild any figure from the existing log file (specific experiment results) by running a ``make`` command from the root project directory (``align-BDD``). For example,

.. code:: bash

   make figures/simpl_heuristics.eps  # to rebuild Figure 6, or
   make figures  # to rebuild all figures for the paper

GNU make will consider files' timestamps and re-run the experiments as necessary (e.g., if the instance generation procedure has changed). To force a complete run (generate all the data, make all experiments, and build all figures), just clean up the previous results first:

.. code:: bash

   make fresh  # to delete all instances and run logs
   make figures  # to rebuild everything

Note that depending on the parameters (which are set in ``Makefile``) and the hardware, it might take quite some time to recalculate everything. PBS scripts [#]_ that were used to run our experiments on the computation cluster are kept in ``📁 pbs`` .

All the recipies are implemented in ``Makefile`` (that's the default makefile name), which is basically a list of commands and source files needed to build each target.  

.. [#] obviously, they are dependent on specific software and hardware on the computation system you use.

Problem instances and raw data
---------------------------------
All the source data is available in the public repository. In particular,

* 📁 `instances` containts all the instance ("raw") data in a compressed format (`tar.gz`; see e.g. `here <https://www.howtogeek.com/362203/what-is-a-tar.gz-file-and-how-do-i-open-it/>`_ if unsure how to open it),
* 📁 `run_logs` contains all the numerical results ("run logs" as CSV files)

The computations are summarized in the following table (scrollable right):

+----------------+---------------------------------------------+-------------------------------------------------------------------+-----------------------------------+--------+-----------------------------+------------------------------------+
| Figure / Table | Filename                                    | Scripts (experiment code)                                         | Raw instances                     | Format | Experiment run log          | Plotting code                      |
+================+=============================================+===================================================================+===================================+========+=============================+====================================+
| Table 1        | ``guessing.tex``                            | :py:mod:`gen_BDD_pair` + :py:mod:`solve_inst`                     | ``orig_problem.tar.gz``           | BDD    | ``main_rnd_run.csv``        | ``tab_guessing.R``                 |
+----------------+---------------------------------------------+                                                                   +                                   +        +                             +------------------------------------+
| Figure 6       | ``simpl_heuristics.eps``                    |                                                                   |                                   |        |                             | ``fig_simpl_heuristics.R``         |
+----------------+---------------------------------------------+                                                                   +                                   +        +                             +------------------------------------+
| Figure 7       | ``orig_obj_histograms.eps``                 |                                                                   |                                   |        |                             | ``fig_obj_hist.R``                 |
+----------------+---------------------------------------------+-------------------------------------------------------------------+                                   +        +-----------------------------+------------------------------------+
| Figure 19      | ``LB.eps``                                  | :py:mod:`gen_BDD_pair` + :py:mod:`experiments.compare_simpl_LBs`  |                                   |        | ``simpl_LB.csv``            | ``fig_LBs.R``                      |
+----------------+---------------------------------------------+-------------------------------------------------------------------+-----------------------------------+        +-----------------------------+------------------------------------+
| Figure 20      | ``orig_lwidth_stats.eps``                   | :py:mod:`gen_BDD_pair` + :py:mod:`experiments.gen_lsizes_stats`   | ``orig_stats.tar.gz``             |        | ``lwidths.csv``             | ``fig_summary.R``                  |
+----------------+---------------------------------------------+-------------------------------------------------------------------+-----------------------------------+        +-----------------------------+------------------------------------+
| Figure 8       | ``orig_runtimes.eps``                       | :py:mod:`gen_BDD_pair` + :py:mod:`experiments.par_scal_test`      | ``orig_scal.tar.gz``              |        | ``orig_scal.csv``           | ``fig_scal.R``                     |
+----------------+---------------------------------------------+-------------------------------------------------------------------+-----------------------------------+        +-----------------------------+------------------------------------+
| Figure 21      | ``no_opts.eps``                             | :py:mod:`gen_BDD_pair` + :py:mod:`experiments.heu_sol_struct`     | ``heu_sol_struct.tar.gz``         |        | ``simpl_sol_struct.csv``    | ``fig_simpl_opt_struct.R``         |
+                +                                             +                                                                   +                                   +        +                             +                                    +
| Figure 22      | ``opts_diam.eps``                           |                                                                   |                                   |        |                             |                                    |
+                +                                             +                                                                   +                                   +        +                             +                                    +
| Figure 23      | ``heuristic_simscore.eps``                  |                                                                   |                                   |        |                             |                                    |
+                +                                             +                                                                   +                                   +        +                             +                                    +
| Figure 24      | ``heuristic_simscore_vs_AB_simscore. eps``  |                                                                   |                                   |        |                             |                                    |
+----------------+---------------------------------------------+-------------------------------------------------------------------+-----------------------------------+--------+-----------------------------+------------------------------------+
| Figure 12      | ``tUFLP_runtimes_overview.eps``             | :py:mod:`experiments.tUFLP_runtimes`                              | ``tUFLP_scal_inst.json``          | json   | ``tUFLP_runtimes_scal.csv`` | ``fig_tUFLP_runtimes_scal.R``      |
+----------------+---------------------------------------------+                                                                   +-----------------------------------+        +-----------------------------+------------------------------------+
| Figure 25      | ``tUFL_runtimes_breakdown.eps``             |                                                                   | ``tUFLP_steps_breakdown.json``    |        | ``tUFLP_runtimes.csv``      | ``fig_tUFLP_runtimes_breakdown.R`` |
+----------------+---------------------------------------------+-------------------------------------------------------------------+-----------------------------------+        +-----------------------------+------------------------------------+
| Figure 26a     | ``various_simpl_vs_min.eps``                | :py:mod:`experiments.tUFL_hist_sizes_control`                     | ``tUFLP_nat_instances.json``      |        | ``tUFLP_nat.csv``           | ``fig_simpl_vs_min.R``             |
+----------------+                                             +                                                                   +-----------------------------------+        +-----------------------------+                                    +
| Figure 26b     |                                             |                                                                   | ``tUFLP_rnd_instances.json``      |        | ``tUFLP_rnd.csv``           |                                    |
+----------------+                                             +-------------------------------------------------------------------+-----------------------------------+--------+-----------------------------+                                    +
| Figure 26c     |                                             | :py:mod:`experiments.rnd_dia_hist_sizes_control`                  | ``bm_heu_random_diagrams.tar.gz`` | BDD    | ``tUFLP_rnd.csv``           |                                    |
+----------------+---------------------------------------------+-------------------------------------------------------------------+-----------------------------------+--------+-----------------------------+------------------------------------+

Figure 18: sample B&B tree was generated by :py:mod:`experiments.sample_BB_tree`.

(Figures 1-5, 9-11, and 13-17 do not involve any computations.)

All the key parameters (instance sizes, generation parameters, etc.) are set in ``Makefile``.

Raw data format
^^^^^^^^^^^^^^^
**BDD** is a structured plain-text description of a BDD explained in more detail in :py:func:`BDD.BDD.save`. Any diagram (say, from file ``filename.bdd``) can be loaded for inspection with the code like:

.. code:: python

          import BDD

          A = BDD.BDD()
          A.load("./filename.bdd")
          A.show()  # will pop up the diagram if the relevant software is installed


**json** describes a tUFLP instance per line of a plain text file. E.g., consider the instance (text line):

    {"S": [[1, 3, 5, 7, 8, 12, 15, 17, 18, 19], [2, 3, 4, 5, 7, 15, 20], [3, 1, 2, 8, 12, 13, 15, 18, 19], [4, 2, 5, 7, 8, 15, 16, 19], [5, 1, 2, 4, 8, 11, 14, 16, 20], [6, 8], [7, 1, 2, 4, 8, 9, 14, 20], [8, 1, 3, 4, 5, 6, 7, 11, 19], [9, 7, 13, 20], [10, 11, 12, 14], [11, 5, 8, 10, 13, 17, 19], [12, 1, 3, 10], [13, 3, 9, 11, 14, 15, 16, 20], [14, 5, 7, 10, 13, 16, 20], [15, 1, 2, 3, 4, 13, 19], [16, 4, 5, 13, 14, 19], [17, 1, 11, 19], [18, 1, 3], [19, 1, 3, 4, 8, 11, 15, 16, 17, 20], [20, 2, 5, 7, 9, 13, 14, 19]], "f": {"1": 9, "2": 6, "3": 6, "4": 8, "5": 8, "6": 8, "7": 6, "8": 7, "9": 6, "10": 9, "11": 7, "12": 6, "13": 5, "14": 8, "15": 7, "16": 8, "17": 6, "18": 5, "19": 9, "20": 7}, "fc": [1, 1, 0, 0, 0, 0, 1, 0, 0, 1, 0, 0, 1, 0, 0, 0, 1, 0, 0, 0], "kb": [5, 2]}

It can be loaded for inspection from a string ``instance`` as follows.

.. code:: python

          import json
          import tUFLP

          inst = json.loads(instance)

          # This will draw corresponding network structure:
          tUFLP.draw_problem_dia(inst["S"], inst["f"],
                                 inst["fc"], inst["kb"])  

After this, dictionary ``inst`` will contain:

- ``inst["S"]``: list of adjacency lists, :math:`S_i` (:math:`M` lists),
- ``inst["f"]``: list of location costs (of length :math:`M`)
- ``inst["fc"]``: list of types assigned to each point, (list of :math:`M` numbers / type IDs)
- ``inst["kb"]``: list of budgets per type (list of :math:`K` numbers)

Code organization
-----------------
The code is organized into several blocks as follows.

Producing figures
^^^^^^^^^^^^^^^^^
The code that physically produces figures from CSV run logs is written in `R <https://www.r-project.org/>`_ (and the wonderful tidyverse/`ggplot2 <https://ggplot2.tidyverse.org>`_ library). The code is conceptually simple and, hopefully, self-explanatory -- see ``.R`` - files in ``📁 post_processing``.

Key data structures and algorithms.
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
The core data structures critical to the paper are presented in the following modules (click on the links for more implementation details and the source code):

.. autosummary:: 
   :toctree: _autosummary
   :recursive:

   BDD
   varseq
   heuristics
   BB_search

Numerical experiments
^^^^^^^^^^^^^^^^^^^^^
Description for specific problems serves the basis for further numerical experiments, which are implemented as separate python programs in ``📁 experiments``

.. autosummary::
   :toctree: _autosummary
   :recursive:

   gen_BDD_pair
   solve_inst
   UFL
   tUFLP
   experiments


Testing framework
^^^^^^^^^^^^^^^^^
Key implementations are covered with tests, mostly using `pytest <https://pytest.org>`_ framework (the tests are in ``📁 tests``).

.. autosummary:: 
   :toctree: _autosummary
   :recursive:

   BDD_test
   BDD_equivalence
   varseq_test
   BB_search_test
   UFLP_test
   tUFLP_test
