How to contribute
=================

Here a brief guide to contributing to this tool:

- fork it on GitHub and create your own repository;

- install it using the "editable" installation or clone it locally on your machine (`virtualenv <https://virtualenv.pypa.io/en/latest/installation.html>`_ usage is strongly suggested);

  .. code:: bash

      $ # pip "editable" installation
      $ pip install -e git+https://github.com/YOUR_USERNAME/ripe-atlas-monitor.git#egg=ripe-atlas-monitor
      
      $ # manual cloning from GitHub (you have to care about dependencies)
      $ git clone https://github.com/YOUR_USERNAME/ripe-atlas-monitor.git
      $ export PYTHONPATH="/path/to/your/ripe-atlas-monitor"

- run the tests in order to be sure that everything is fine;

  .. code:: bash

      $ nosetests -vs

- finally, start making your changes and, possibly, add test units and docs to make the merging process easier.

Once you have done, please run tests again:

.. code:: bash

    $ tox

If everything is fine, push to your fork and `create a pull request <https://help.github.com/articles/using-pull-requests/>`_.

Code
----

Adding a new check
++++++++++++++++++

..
        Keep in sync with
        - ParsedResults.py/ParsedResult class docstring
        - ExpResCriteriaBase.py/ExpResCriterion class docstring
        - Analyzer.py/BasePropertyAnalyzer class docstring
        - Analyzer.py/BaseResultsAnalyzer class docstring

- **ParsedResults.py**: a new property must be handled by a ``ParsedResult`` class in order to parse and prepare the new
  result to be checked.
  Create a new class (if needed) and add a new property (``PROPERTIES`` and ``@property``).
  The ``prepare()`` method must call ``self.set_attr_to_cache()`` to store the parsed values;
  the ``@property`` must read the value using ``self.get_attr_from_cache()`` and return it.
  More info on the ``ParsedResult`` class docstring.

- **ExpResCriteriaXXX.py**: an ``ExpResCriterion``-derived class must implement the ``__init__()`` method to 
  read the expected result attributes from the monitor's configuration and validate them.
  The ``prepare()`` method must parse the result received from probes and store its value in a local attribute
  (something like ``self.response_xxx``); the ``result_matches()`` method must compare the parsed result 
  received from the probe with the expected result.
  The new class must be added to the appropriate list in **ExpResCriteria.py**.
  More info on the ``ExpResCriterion`` class docstring (**ExpResCriteriaBase.py**). See also :ref:`expres_classes_docstring`.

- **tests/monitor_cfg_test.py**:

  - Add the new criterion to ``self.criteria`` in ``TestMonitorCfg.setUp()``
    (at least the ``CRITERION_NAME`` with a correct value).

  - Add a call to ``self.add_criterion("<criterion_name>")`` to the ``test_expres_XXX()`` methods, where *XXX* is
    the measurements' type this expected result applies to.

  - Add some tests for the new expected result configuration syntax.

- **tests/doc_test.py**: if the new expected result contains lists attributes, add the configuration field name to the 
  appropriate ``exp_list_fields`` variables.

- **tests/results_XXX_test.py**: add some tests to validate how the new criterion processes results from real measurements.

Adding a new report to the analyzer
+++++++++++++++++++++++++++++++++++

- **ParsedResults.py**: a new class must be created (or a new property added to an existing ``ParsedResult``-derived class). See the previous section.

- **Analyzer.py**:
 
  - This step can be avoided if the new ``ParsedResult`` property to analyze is handled by an already existing ``ParsedResult`` class.

    Create a new ``BaseResultsAnalyzer``-derived class; its ``get_parsed_results()`` method must yield each 
    ``ParsedResult`` element that need to be analyzed (for example, the result itself or each DNS response 
    for DNS measurements' results). The ``BaseResultsAnalyzer`` and ``ResultsAnalyzer_DNSBased`` classes 
    should already do most of the work. Add the new class to the ``Analyzer.RESULTS_ANALYZERS`` list.

  - The ``BaseResultsAnalyzer.PROPERTIES_ANALYZERS_CLASSES`` and ``BaseResultsAnalyzer.PROPERTIES_ORDER`` lists
    must contain the new property defined in the ``ParsedResult``-derived class.

  - A ``BasePropertyAnalyzer``-derived class must be created in order to implement the aggregation mechanism and
    the output formatting for the new property. More info on the ``BasePropertyAnalyzer`` and ``BaseResultsAnalyzer`` classes docstring.

- **docs/COMMANDS.rst**: add the new property to the list of those supported by the ``analyze`` command.

Data for unit testing
+++++++++++++++++++++

The **tests/data/download_msm.py** script can be used to download and store data used for tests. It downloads measurement's metadata, results and probes' information and stores them in a **measurement_id**.json file. The **tests/data.py** module loads the JSON files that can subsequently be used for unit testing purposes.

.. _expres_classes_docstring:

ExpResCriterion-derived classes docstring
+++++++++++++++++++++++++++++++++++++++++

These classes must use a special syntax in their docstrings which allows to automatically build documentation and config example (**doc.py** ``build_doc`` and ``build_monitor_cfg_tpl`` functions).

Example::

        Criterion: rtt

            Test the median round trip time toward destination.

            Available for: ping, traceroute.

            `rtt`: maximum RTT (in ms).

            `rtt_tolerance` (optional): tolerance (in %) on `rtt`.

            If `rtt_tolerance` is not given, match when measured RTT is less
            than `rtt`, otherwise match when measured RTT is within `rtt`
            +/- `rtt_tolerance` %.

            Examples:

            expected_results:
              LowRTT:
                rtt: 50
              Near150:
                rtt: 150
                rtt_tolerance: 30

- The first line must include only the "Criterion: xxx" string, where *xxx* is the class ``CRITERION_NAME`` attribute.

  Example: ``Criterion: rtt``

- A brief description of the expected result must follow.

  Example: ``Test the median round trip time toward destination.``

- The list of measurements' types for which this expected result can be used must follow, in the format ``Available for: x[, y[, z]].``, where values are valid measurements' types (``ping``, ``traceroute``, ...).

  Example: ``Available for: ping, traceroute.``

- A list of configuration fields must follow. Every docstring line starting with a backquote is considered to be a field name.

  The format must be the following:

  ```field_name` ["(optional)"]: ["list"] "description..."``

  The "(optional)" string is used to declare this field as optional, otherwise it's considered mandatory.

  The "list" string is used to declare that this field contains a list of values.

  Example: ```rtt`: maximum RTT (in ms).``, ```rtt_tolerance` (optional): tolerance (in %) on `rtt`.``, ```dst_ip`: list of expected IP addresses (or prefixes).``

- A (long) description of how this expected result's fields are used can follow. Here, be careful to avoid lines starting with the backquote, otherwise they will be interpreted as a field declaration.

- Finally, a line starting with the "Example" or "Examples" strings can be used to show some examples. They will be formatted using code blocks.
