.. _testing:

Testing
=======

We maintain a series of tests in Trident to make sure the code gives consistent
results and to catch accidental breakages in our source code and dependencies.
These tests are run by `Travis <https://travis-ci.org/>`_ automatically and 
regularly to assure consistency in functionality, but you can run them locally
too (see below).  The tests consist of a mix of unit tests (tests to assure Trident 
functions don't actively fail) and answer tests (tests comparing newly 
generated results against some old established results to assure consistency).

.. _running-the-tests:

Running the Test Suite
----------------------

Running the test suite requires a version of Trident installed from
source (see :ref:`install-dev`).

The tests are run using the ``pytest`` Python module.  This can be
installed with ``conda`` or ``pip``.

.. code-block:: bash

   $ conda install pytest

The test suite requires a number of datasets for testing functionality.
Trident comes with a helper script that will download all the datasets and 
untar them.  Before running this, make sure you have the 
``answer_test_data_dir`` variable set in your config file (see :ref:`step-3`).  
This variable should point to a directory where these datasets will be stored.  
The helper script is located in the ``tests`` directory of the Trident source.

.. code-block:: bash

   $ cd tests
   $ python download_test_data.py

If this is your first time running the tests, then you need to generate a
"gold standard" for the answer tests. Follow :ref:`generating-answer-tests` 
before continuing with running the tests, otherwise your answer tests will 
fail.

Make sure you're on the desired version of yt and trident that you want to 
test and use (usually the tip of the development branch i.e., ``master``).  

.. code-block:: bash

   $ export TRIDENT_GENERATE_TEST_RESULTS=0
   $ cd /path/to/yt/
   $ git checkout master
   $ pip install -e .
   $ cd /path/to/trident
   $ git checkout master
   $ pip install -e .

The test suite is run by calling ``py.test`` from within the ``tests`` 
directory.

.. code-block:: bash

   $ cd tests
   $ py.test
   ============================= test session starts ==============================
   platform darwin -- Python 3.6.0, pytest-3.0.7, py-1.4.32, pluggy-0.4.0
   rootdir: /Users/britton/Documents/work/yt/extensions/trident/trident, inifile:
   collected 52 items

   test_absorption_spectrum.py ..........
   test_download.py .
   test_generate.py .
   test_instrument.py .
   test_ion_balance.py ............
   test_light_ray.py .....
   test_line_database.py .......
   test_lsf.py ....
   test_pipelines.py ...
   test_plotting.py .
   test_ray_generator.py .
   test_spectrum_generator.py ......

   ========================= 52 passed in 117.32 seconds ==========================

If a test fails for some reason, you will be given a detailed traceback and
reason for it failing.  You can use this to identify what is wrong with your
source or perhaps a change in the code of your dependencies.  The tests should
take less than five minutes to run.

.. _generating-answer-tests:

Generating Test Results
-----------------------

In order to assure the Trident codebase gives consistent results over time, 
we compare the outputs of tests of new versions of Trident against an older, 
vetted version of the code we think gives accurate results.  To create this
"gold standard" result from the older version of the code, you must roll back 
the Trident and yt source back to the older "trusted" versions of the code.  
You can find the tags for the most recent trusted versions of the code by 
running ``gold_standard_versions.py`` and then rebuilding yt and Trident 
with these versions of the code.  Lastly, set the 
``TRIDENT_GENERATE_TEST_RESULTS`` environment variable to 1 and run the tests:

.. code-block:: bash

   $ cd tests
   $ python gold_standard_versions.py
   
   Latest Gold Standard Commit Tags
   yt = 38b79c094ca9
   Trident = test-standard-v1

   To update to them, `git checkout <tag>` in appropriate repository

   $ cd /path/to/yt
   $ git checkout 38b79c094ca9
   $ pip install -e .
   $ cd /path/to/trident
   $ git checkout test-standard-v1
   $ pip install -e .
   $ export TRIDENT_GENERATE_TEST_RESULTS=1
   $ cd tests
   $ py.test

The test results should now be stored in the ``answer_test_data_dir`` that
you specified in your Trident configuration file. You may now run the actual 
tests (see :ref:`running-the-tests`) with your current version of yt and 
Trident comparing against these gold standard results.

.. _updating-the-test-results:

Updating the Test Results
--------------------------

Periodically, the gold standard for our answer tests must be updated as bugs 
are caught or new more accurate behavior is enabled that causes the answer
tests to fail.  The first thing to do
is to identify the most accurate version of the code (e.g., changesets for 
yt and trident that give the desired behavior).  Tag the Trident changeset with
the next gold standard iteration.  You can see the current iteration by looking
in the ``.travis.yml`` file at the ``TRIDENT_GOLD`` entry--enumerate this and
tag the changeset.  Update the ``.travis.yml`` file so that the ``YT_GOLD`` and
``TRIDENT_GOLD`` entries point to your desired changeset and tag.  You have to 
explicitly push the new tag (hereafter ``test-standard-v2``) to 
the repository.

.. code-block:: bash

   $ git tag test-standard-v2 <trident-changeset>
   $ ... edit .travis.yml files to update YT_GOLD=<yt changeset>
   $ ... and TRIDENT_GOLD=<test-standard-v2
   $ git add .travis.yml
   $ git commit
   $ git push origin
   $ git push origin test-standard-v2

Lastly, someone with admin access to the main trident repository will have to 
clear Travis' cache, so that it regenerates new answer test results.  This can 
be done manually here: https://travis-ci.org/trident-project/trident/caches .
