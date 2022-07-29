Implementation: overview
------------------------

These pages document the implementation of the numerical experiments for
"Align-BDD" working paper by Bochkarev and Smith. The notes along with the
accompanying `source code and data <https://github.com/alex-bochkarev/align-BDD>`_
are assumed to provide enough information to:

(1) inspect the raw data,
(2) reproduce / adjust / inspect the figures from the paper (given the experiment log files) as necessary, and
(3) completely reproduce the paper results from scratch, including instance generation, with reasonable efforts.

It is our special interest to help the reader build upon our results, so please
do not hesitate to address any technical questions to `Alexey
Bochkarev <https://www.bochkarev.io/contact>`_.

Computational infrastructure
----------------------------

To run the experiments we used machines running GNU/Linux system (remote
computational resources provided by `Clemson Palmetto cluster
<https://www.palmetto.clemson.edu/palmetto/about/>`_ and laptops), running the
following software:

* experiments implementation: python3. :download:`packages list <requirements.txt>`, the usual way should work::

    pip install -r requirements.txt

* figures: R. To install (a superset of) the necessary libraries, from within R session:

.. code-block:: R

          install.packages(c("dplyr", "ggplot2", "ggridges",
             "gridExtra", "latex2exp", "optparse", "reshape",
             "stringr", "tidyr", "viridis", "xtable"))

* parallel computation: `GNU parallel <https://www.gnu.org/software/parallel/>`_,
* experiments pipeline: GNU make and PBS (for job scheduling on the cluster).
* (we also used a separate `tool <https://github.com/alex-bochkarev/tgs-curl>`_ to receive the results promptly via `Telegram <https://telegram.org>`_, but this can be safely removed)
* numerical experiments require `Gurobi <https://www.gurobi.com>`_ solver.
* most graphs were visualized used a stand-alone program called `Graphviz <https://graphviz.org/>`_
  Note that before ``pip install pygraphviz`` you might need to install the program
  and the header files as well. On my Debian distribution I needed to do
  ``sudo apt install graphviz graphviz-dev``.

Note that for the (python) environment management we used the recommended
`venv-way <https://docs.python.org/3/library/venv.html>`_. For example, the
virtual environment can be created and activated with:

.. code-block:: bash

                python -m venv .venv
                source .venv/bin/activate

                # so that we can install the necessary packages there with:
                pip install -r requirements.txt

(``conda`` would work just as well.)

General reproducibility
-----------------------

Assuming the necessary software is present, one can rebuild any figure from the
existing log file (specific experiment results) by running a ``make`` command
from the root project directory (``align-BDD``). For example, theoretically this
will work:

.. code:: bash

   make figures/simpl_heuristics.eps  # to rebuild Figure 6, or
   make figures  # to rebuild all figures for the paper

GNU make will consider files' timestamps and re-run the
experiments as necessary (e.g., if the instance generation procedure has
changed). To force a complete run (generate all the data, make all experiments,
and build all figures), just clean up the previous results first:

.. code:: bash

   make fresh  # to delete all instances and run logs
   make figures  # to rebuild everything

Note, however, that depending on the parameters (which are set in ``Makefile``)
and the hardware, it might take quite some time to recalculate everything. PBS
scripts that were used to run our experiments on the computation cluster are
kept in folder ``📁 pbs``. Certainly, they are dependent on the hardware and the
available software, but these materials hopefully will provide a good starting
point to reproduce the results presented in the paper.

All the recipies (how to make each figure or other artifact) are described in
the file called ``Makefile``, which is basically a list of commands and source
files needed to build each target.

Project repository structure.
-----------------------------
All the source data is available in the `project repository <https://github.com/alex-bochkarev/align-BDD>`_.

Key folders are:

* ``📁 instances``: all the instance ("raw") data in a compressed format (`tar.gz`; see e.g. `here <https://www.howtogeek.com/362203/what-is-a-tar.gz-file-and-how-do-i-open-it/>`_ if unsure how to open it),
* ``📁 run_logs``: all the numerical results -- "run logs," mostly as comma separated files (``.csv``)
* ``📁 experiments`` and the root folder contain all key implementation files (in Python).
* ``📁 post_processing``: the R code that produces figures from the run logs.
* ``📁 figures``: the key figures themselves.
* ``📁 docs``: the code documentation source (i.e., these pages).
* ``📁 tests``: most of the code testing (for python code).
* ``📁 pbs``: auxiliary `PBS <https://en.wikipedia.org/wiki/Portable_Batch_System>`_ scripts used to run the code on the computational cluster.

The computations are summarized in the following table (scrollable right):

+----------------+---------------------------------------------+-------------------------------------------------------------------+-----------------------------------+--------+-----------------------------+------------------------------------+
| Figure / Table | Filename                                    | Experiments code (scripts)                                        | Raw instances                     | Format | Experiment run log          | Plotting code                      |
+================+=============================================+===================================================================+===================================+========+=============================+====================================+
| Table 1        | ``guessing.tex``                            | :py:mod:`gen_BDD_pair` + :py:mod:`solve_inst`                     | ``orig_problem.tar.gz``           | BDD    | ``main_rnd_run.csv``        | ``tab_guessing.R``                 |
+----------------+---------------------------------------------+                                                                   +                                   +        +                             +------------------------------------+
| Figure 6       | ``simpl_heuristics.eps``                    |                                                                   |                                   |        |                             | ``fig_simpl_heuristics.R``         |
+----------------+---------------------------------------------+                                                                   +                                   +        +                             +------------------------------------+
| Figure 7       | ``orig_obj_histograms.eps``                 |                                                                   |                                   |        |                             | ``fig_obj_hist.R``                 |
+----------------+---------------------------------------------+-------------------------------------------------------------------+                                   +        +-----------------------------+------------------------------------+
| Figure 17      | ``LB.eps``                                  | :py:mod:`gen_BDD_pair` + :py:mod:`experiments.compare_simpl_LBs`  |                                   |        | ``simpl_LB.csv``            | ``fig_LBs.R``                      |
+----------------+---------------------------------------------+-------------------------------------------------------------------+-----------------------------------+        +-----------------------------+------------------------------------+
| Figure 18      | ``orig_lwidth_stats.eps``                   | :py:mod:`gen_BDD_pair` + :py:mod:`experiments.gen_lsizes_stats`   | ``orig_stats.tar.gz``             |        | ``lwidths.csv``             | ``fig_summary.R``                  |
+----------------+---------------------------------------------+-------------------------------------------------------------------+-----------------------------------+        +-----------------------------+------------------------------------+
| Figure 8       | ``orig_runtimes.eps``                       | :py:mod:`gen_BDD_pair` + :py:mod:`experiments.par_scal_test`      | ``orig_scal.tar.gz``              |        | ``orig_scal.csv``           | ``fig_scal.R``                     |
+----------------+---------------------------------------------+-------------------------------------------------------------------+-----------------------------------+        +-----------------------------+------------------------------------+
| Figure 19      | ``no_opts.eps``                             | :py:mod:`gen_BDD_pair` + :py:mod:`experiments.heu_sol_struct`     | ``heu_sol_struct.tar.gz``         |        | ``simpl_sol_struct.csv``    | ``fig_simpl_opt_struct.R``         |
+                +                                             +                                                                   +                                   +        +                             +                                    +
| Figure 20      | ``opts_diam.eps``                           |                                                                   |                                   |        |                             |                                    |
+                +                                             +                                                                   +                                   +        +                             +                                    +
| Figure 21      | ``heuristic_simscore.eps``                  |                                                                   |                                   |        |                             |                                    |
+                +                                             +                                                                   +                                   +        +                             +                                    +
| Figure 22      | ``heuristic_simscore_vs_AB_simscore. eps``  |                                                                   |                                   |        |                             |                                    |
+----------------+---------------------------------------------+-------------------------------------------------------------------+-----------------------------------+--------+-----------------------------+------------------------------------+
| Figure 9b      | ``tMIP_tMIPCPP_tDD.eps``                    | :py:mod:`UFLP_2_cav`  :py:func:`darkcloud.gen_caveman_inst`       | ``jUFLP.tar.gz``                  | json   | ``2022-07-19_jUFLP.csv``    | ``fig_DDvsMIP.R``                  |
+                +                                             +                                                                   +                                   +        +                             +                                    +
| Figure 10a     | ``intDD_VS_vs_toA.eps``                     | :py:mod:`jUFLP_cavemen`  :py:mod:`jUFLP_utils`                    |                                   |        |                             |                                    |
+                +                                             +                                                                   +                                   +        +                             +                                    +
| Figure 10b     | ``tMIP_tMIPCPP_tDD.eps``                    | :py:func:`experiments.softcover.generate_overlaps`                |                                   |        |                             |                                    |
+----------------+---------------------------------------------+-------------------------------------------------------------------+-----------------------------------+--------+-----------------------------+------------------------------------+

Figure 16: sample B&B tree was generated by :py:mod:`experiments.sample_BB_tree`.

(Figures 1-5, 9a, 11-15, and 23-26 do not involve any computations.)

All the key parameters (instance sizes, generation parameters, etc.) are set in ``Makefile``.

Implementation details
----------------------

Here are implementations of the key algorithms from the paper:

+-------------+------------------------------------------------------------+-------------------------------------------------------------------------------+
| Reference   | Brief Description                                          | Implementation                                                                |
+=============+============================================================+===============================================================================+
| Sec. 2.1    | BDD data structure implementation                          | :py:class:`BDD.BDD`                                                           |
+-------------+------------------------------------------------------------+-------------------------------------------------------------------------------+
| Sec. 3.2    | Weighted variable sequence (``VarSeq``) implementation.    | :py:class:`varseq.VarSeq`                                                     |
+-------------+------------------------------------------------------------+-------------------------------------------------------------------------------+
| Algo 3      | A ``swap`` operation for a BDD.                            | :py:func:`BDD.BDD.swap_up`                                                    |
+-------------+------------------------------------------------------------+-------------------------------------------------------------------------------+
| Algo 4      | | Align-to (weighted variable sequence, linear time)       | | :py:func:`varseq.VarSeq.align_to`                                           |
|             | | (Algo 1 is slower and is not used in the experiments)    | | (see also :py:func:`varseq.VarSeq.greedy_sort`)                             |
+-------------+------------------------------------------------------------+-------------------------------------------------------------------------------+
| Algo 5      | | Branch-and-bound algorithm for the simplified problem.   | | See :py:class:`BB_search.BBSearch` and :py:func:`BB_search.BBSearch.search` |
|             | | - lower bound:                                           | | :py:func:`BB_search.SearchNode.calculate_LB`                                |
|             | | - upper bound:                                           | | :py:func:`BB_search.SearchNode.calculate_UB`                                |
+-------------+------------------------------------------------------------+-------------------------------------------------------------------------------+
| Sec. 4.1.2  | Various heuristics for aligning BDDs and VarSeqs.          | :py:mod:`heuristics`                                                          |
+-------------+------------------------------------------------------------+-------------------------------------------------------------------------------+
| Algo 6      | | Creating a BDD encoding an UFLP sub-instance.            | :py:func:`UFLP_fullDD.create_cover_DD`                                        |
|             | | (for a given order of variables)                         |                                                                               |
+-------------+------------------------------------------------------------+-------------------------------------------------------------------------------+
| | Example 5 | | Finding a good order of variables                        | :py:func:`UFLPOrder.UFLP_greedy_order`                                        |
| | (App. H)  | | for an UFLP sub-instance.                                |                                                                               |
+-------------+------------------------------------------------------------+-------------------------------------------------------------------------------+
| Sec. 4.2    | | Solving j-UFLP:                                          | | See :py:mod:`UFLP_2_cav` for the overview, and then, in particular:         |
|             | | - with naive MIP (formulation (2)):                      | | :py:func:`jUFLP_cavemen.solve_cm_jUFLP_MIP`                                 |
|             | | - CPP as an MIP:                                         | | :py:func:`jUFLP_cavemen.solve_cm_jUFLP_CPPMIP_fullDDs`                      |
|             | | - CPP / shortest-path over intersection BDD:             | | :py:func:`jUFLP_cavemen.solve_cm_jUFLP_fullDDs`                             |
+-------------+------------------------------------------------------------+-------------------------------------------------------------------------------+

Raw data formats
----------------

Binary Decision Diagrams
^^^^^^^^^^^^^^^^^^^^^^^^

A ``.bdd`` file represents a structured plain-text description of a BDD
 explained in more detail in :py:func:`BDD.BDD.save`. Any diagram (say, from
 file ``filename.bdd``) can be loaded for inspection with the code like:

.. code:: python

          import BDD

          A = BDD.BDD()
          A.load("./filename.bdd")
          A.show()  # will pop up the diagram if the relevant software is installed


Note that each align-BDD instance is just a pair of BDDs with the same number.
For example, instance number one from ``orig_problem.tar.gz`` (which ends up,
for example, in Figure 7) is contained in two files, ``A1.bdd`` and ``B1.bdd``.


j-UFLP instance data
^^^^^^^^^^^^^^^^^^^^

A ``.json`` file describes a j-UFLP instance in `JSON format <https://docs.python.org/3/library/json.html>`_.
For more details on how exactly it is saved, see :py:func:`jUFLP_utils.save_inst`.

Each instance can be loaded for inspection from a file as follows, using the
auxiliary code we have in :py:mod:`jUFLP_utils`:

.. code:: python

          import json
          from jUFLP_utils import load_inst, draw_jUFLP_inst

          i1, i2, link = load_inst("./instances/jUFLP_cm/inst_1.json")

          draw_jUFLP_inst(i1, i2, link, filename="tmp/jUFLP-check.dot")

After this, variables ``i1``, ``i2``, and ``link`` will contain the j-UFLP
instance description (respectively, the two sub-instances and the linking
dictionary). Moreover, function :py:func:`jUFLP_utils.draw_jUFLP_inst` will
create a ``.dot`` (Graphviz) file visualizing the instance as a graph,
similar to Figure 9a. So that invoking ``dot -Tx11 tmp/jUFLP-check.dot``
(on my GNU/Linux machine) gives something like:

.. image:: ./jUFLP_sample.png
  :width: 600
  :alt: An example of j-UFLP instance (two sub-instances)


Code organization
-----------------
The code is organized into several blocks (modules) as follows.

Producing figures
^^^^^^^^^^^^^^^^^

The code that physically produces figures from CSV run logs is written in `R
<https://www.r-project.org/>`_ (and the wonderful tidyverse/`ggplot2
<https://ggplot2.tidyverse.org>`_ library). The code is conceptually simple and,
hopefully, self-explanatory -- see ``.R`` - files in ``📁 post_processing``.

Key data structures and algorithms.
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The core data structures critical to the paper are presented in the following
modules (click on the links for more implementation details and the source
code):

.. autosummary::
   :toctree: _autosummary
   :recursive:

   BDD
   varseq
   heuristics
   BB_search

The code relevant to j-UFLP application (Section 4.2) is here:

.. autosummary::
   :toctree: _autosummary
   :recursive:

   UFLP_fullDD
   UFLPOrder
   jUFLP_cavemen
   jUFLP_utils

Numerical experiments
^^^^^^^^^^^^^^^^^^^^^

Description for specific problems serves the basis for further numerical
experiments, which are implemented as separate python programs in ``📁
experiments``

.. autosummary::
   :toctree: _autosummary
   :recursive:

   gen_BDD_pair
   solve_inst
   experiments
   UFLP_2_cav
   darkcloud

Testing framework
^^^^^^^^^^^^^^^^^

Key implementations are covered with tests, mostly using `pytest
<https://pytest.org>`_ framework. On top of the testing code in the files
mentioned above (testing functions have ``test_`` prefix in the name), some
tests are in ``📁 tests``:

.. autosummary::
   :toctree: _autosummary
   :recursive:

   BDD_test
   BDD_equivalence
   varseq_test
   BB_search_test
   UFLP_test
