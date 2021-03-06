Link two datasets
=================

Introduction
------------

This example shows how two datasets with data about persons can be
linked. We will try to link the data based on attributes like first
name, surname, sex, date of birth, place and address. The data used in
this example is part of
`Febrl <https://sourceforge.net/projects/febrl/>`__ and is fictitious.

First, start with importing the ``recordlinkage`` module. The submodule
``recordlinkage.datasets`` contains several datasets that can be used
for testing. For this example, we use the Febrl datasets 4A and 4B.
These datasets can be loaded with the function ``load_febrl4``.

.. ipython::

    In [0]: import recordlinkage
       ...: from recordlinkage.datasets import load_febrl4

The datasets are loaded with the following code. The returned datasets
are of type ``pandas.DataFrame``. This makes it easy to manipulate the
data if desired. For details about data manipulation with ``pandas``,
see their comprehensive documentation http://pandas.pydata.org/.

.. ipython::

    In [0]: dfA, dfB = load_febrl4()
       ...: dfA


Make record pairs
-----------------

It is very intuitive to compare each record in DataFrame ``dfA`` with
all records of DataFrame ``dfB``. In fact, we want to make record pairs.
Each record pair should contain one record of ``dfA`` and one record of
``dfB``. This process of making record pairs is also called "indexing".
With the ``recordlinkage`` module, indexing is easy. First, load the
``index.Index`` class and call the ``.full`` method. This object
generates a full index on a ``.index(...)`` call. In case of
deduplication of a single dataframe, one dataframe is sufficient as
argument.

.. ipython::

    In [0]: indexer = recordlinkage.Index()
       ...: indexer.full()
       ...: pairs = indexer.index(dfA, dfB)


With the method ``index``, all possible (and unique) record pairs are
made. The method returns a ``pandas.MultiIndex``. The number of pairs is
equal to the number of records in ``dfA`` times the number of records in
``dfB``.

.. ipython::

    In [0]: print (len(dfA), len(dfB), len(pairs))

Many of these record pairs do not belong to the same person. In case of
one-to-one matching, the number of matches should be no more than the
number of records in the smallest dataframe. In case of full indexing,
``min(len(dfA), len(N_dfB))`` is much smaller than ``len(pairs)``. The
``recordlinkage`` module has some more advanced indexing methods to
reduce the number of record pairs. Obvious non-matches are left out of
the index. Note that if a matching record pair is not included in the
index, it can not be matched anymore.

One of the most well known indexing methods is named *blocking*. This
method includes only record pairs that are identical on one or more
stored attributes of the person (or entity in general). The blocking
method can be used in the ``recordlinkage`` module.

.. ipython::

    In [0]: indexer = recordlinkage.Index()
       ...: indexer.block("given_name")
       ...: candidate_links = indexer.index(dfA, dfB)
       ...: len(candidate_links)


The argument "given\_name" is the blocking variable. This variable has
to be the name of a column in ``dfA`` and ``dfB``. It is possible to
parse a list of columns names to block on multiple variables. Blocking
on multiple variables will reduce the number of record pairs even
further.

Another implemented indexing method is *Sorted Neighbourhood Indexing*
(``recordlinkage.index.SortedNeighbourhood``). This method is very
useful when there are many misspellings in the string were used for
indexing. In fact, sorted neighbourhood indexing is a generalisation of
blocking. See the documentation for details about sorted neighbourd
indexing.

Compare records
---------------

Each record pair is a candidate match. To classify the candidate record
pairs into matches and non-matches, compare the records on all
attributes both records have in common. The ``recordlinkage`` module has
a class named ``Compare``. This class is used to compare the records.
The following code shows how to compare attributes.

.. ipython::

    In [0]: compare_cl = recordlinkage.Compare()
       ...: compare_cl.exact("given_name", "given_name", label="given_name")
       ...: compare_cl.string("surname", "surname", method="jarowinkler", threshold=0.85, label="surname")
       ...: compare_cl.exact("date_of_birth", "date_of_birth", label="date_of_birth")
       ...: compare_cl.exact("suburb", "suburb", label="suburb")
       ...: compare_cl.exact("state", "state", label="state")
       ...: compare_cl.string("address_1", "address_1", threshold=0.85, label="address_1")
       ...: features = compare_cl.compute(candidate_links, dfA, dfB)

The comparing of record pairs starts when the ``compute`` method is
called. All attribute comparisons are stored in a DataFrame with
horizontally the features and vertically the record pairs.

.. ipython::

    In [0]: features

.. ipython::

    In [0]: features.describe()

The last step is to decide which records belong to the same person. In
this example, we keep it simple:

.. ipython::

    In [0]: features.sum(axis=1).value_counts().sort_index(ascending=False)

.. ipython::

    In [0]: features[features.sum(axis=1) > 3]


Full code
---------

.. code:: ipython3

    import recordlinkage
    from recordlinkage.datasets import load_febrl4
    
    dfA, dfB = load_febrl4()
    
    # Indexation step
    indexer = recordlinkage.Index()
    indexer.block("given_name")
    candidate_links = indexer.index(dfA, dfB)
    
    # Comparison step
    compare_cl = recordlinkage.Compare()
    
    compare_cl.exact("given_name", "given_name", label="given_name")
    compare_cl.string("surname", "surname", method="jarowinkler", threshold=0.85, label="surname")
    compare_cl.exact("date_of_birth", "date_of_birth", label="date_of_birth")
    compare_cl.exact("suburb", "suburb", label="suburb")
    compare_cl.exact("state", "state", label="state")
    compare_cl.string("address_1", "address_1", threshold=0.85, label="address_1")
    
    features = compare_cl.compute(candidate_links, dfA, dfB)
    
    # Classification step
    matches = features[features.sum(axis=1) > 3]
    print(len(matches))


