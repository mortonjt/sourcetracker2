[![Build Status](https://travis-ci.com/biota/sourcetracker2.svg?token=cRee6r8tqQgg7M8jqmie)](https://travis-ci.com/biota/sourcetracker2)

# SourceTracker2

SourceTracker was originally described in [Knights et al., 2011](http://www.ncbi.nlm.nih.gov/pubmed/21765408).
If you use this package, please cite the original SourceTracker paper linked 
above.

# Documentation

This script replicates and extends the functionality of Dan Knights's
SourceTracker R package.

The ``mapping file`` which describes the ``sources`` and ``sinks`` must be
formatted in the same way it was for the SourceTracker R package. Specifically, 
there must be a column ``SourceSink`` and a column ``Env``. For an example, look
at ``sourcetracker2/data/tiny-test/``. 

A major improvment in this version of SourceTracker is the ability to run the
code in parallel. Currently, parallelization across a single machine is
supported for both estimation of source proportions and leave-one-out source
class prediction. The speedup from parallelization should be approximately a 
factor of ``jobs`` that are passed. For instance, passing ``--jobs 4`` should
create a speedup of roughly 4X (less due to overhead). The package 
``ipyparallel`` is used to enable parallelization.

# Theory

This readme describes some of the basic theory for use of SourceTracker2. For
more theory, please see the [juypter notebook](https://github.com/biota/SourceTracker_rc/blob/master/ipynb/Sourcetracking%20using%20a%20Gibbs%20Sampler.ipynb).

There are several ways to use this script:
 (1) Estimating the proportions of different (microbial) sources to a sample of
     a (microbial) sink.
 (2) Using a leave-one-out (LOO) strategy, predict the metadata class of a
     given (microbial) sample.

The main functionality is (1), the estimation of the proportion of `sources`
to a given `sink`. A `source` and a `sink` are both vectors of feature
abundances. A  `source` is usually the sum of multiple samples that come from
an environment of interest, and a `sink` is usually a single sample. As an
example, consider the classic balls in urns problem. There are three urns, each
of which contains a set of differently colored balls in different proportions.

Imagine that you reach into Urn1 and remove ``u_1`` balls without looking at the
colors. You reach into Urn2 and remove ``u_2`` balls, again without looking at
the colors. Finally, you reach into Urn3 and remove ``u_3`` balls, without
looking at the colors. You then mix your individual samples (``u_1``, ``u_2``,
and ``u_3``) and produce one mixed sample whose color counts you survey.

|        | Urn1 | Urn2 | Urn3 | Sample |
|--------|:----:|------|------|--------|
| Blue   |   3  | 6    | 100  | 26     |
| Red    |  55  | 12   | 30   | 9      |
| Green  |  10  | 0    | 1    | 1      |
| ...    |      |      |      |        |
| Orange | 79   | 18   | 0    | 50     |


Your goal is to recover the numbers ``u_1``, ``u_2``, and ``u_3`` using only the
knowledge of the colors of your mixed sample and the proportions of each color
in the sinks.

The Gibb's sampler is a method for estimating ``u_1``, ``u_2`` and ``u_3`` that
is optimal in many senses. In this script, the Gibb's sampler is used to make an
estimate of the source proportions (``u_1``, ``u_2``, and ``u_3``) plus an
estimate of the proportion of balls in the sample that came from an 'unknown'
source. In the urns example there is no unknown source; all the balls came from
one of the three urns. In a real set of microbial samples, however, it is
exceedingly likely that the urns (source samples) that have been assayed are not
the source of every microbe that has been found in the sample (air microbes,
skin microbes of those processing the samples, etc.).

In practice, researchers often take multiple samples from a given source
environment (e.g. to learn the true distribution of features in the source). It
is desirable to 'collapse' samples from the same source into one representative
source sample. This is mainly for interpretability. Consider the urn example
above. In reality, we would not exactly know the contents of any of the urns.
The only way to learn these 'true' source distributions would be to sample them
repeatedly. Combining N samples from urn 1 into a single source would make that
estimate of the urns true proportions of different colors more accurate and
would make interpreting our results easier; we'd have only 3 source proportions
(plus the unknown) to interpret rather than 3N+1 (assuming N samples from each
urn). Please read about (2) below, however, to understand an important
limitation of this collapsing processes.

A second function of of this script is (2), the prediction of the metadata class
of sample based on the feature abundances in all samples and the metadata
classes of all samples.

In practice, this function is useful for checking whether the source groupings
you have computed are good groupings. As an example, imagine that you are baking
bread and want to know where the microbes are coming from in your dough. 
You think there are three main sources: flour, water, hands. You take 10 samples
from each of those environments (10 different bags of flour, 10 samples from
water, 10 samples from your hands on different days). For computing source
proportions, you would normally collapse each of the 10 samples from the given 
class into one source (so you'd end up with a 'Flour', 'Water', and 'Hand'
source). However, if the flour you use comes from different facilities, its very
likely that it will have very different microbial composition. If this is the
case, collapsing the flour samples into a single source would be inappropriate,
since there are really two (at least) different sources of microbes from the
flour. To check the homogenity of your source classifications, you can use the
LOO strategy to make sure that all sources within each class look the same.

# Usage examples

These usage examples expect that you are in the directory
``sourcetracker2/data/tiny-test/``.

Calculate the proportion of each source in each sink.
``sourcetracker2 gibbs -i otu_table.biom -m map.txt -o mixing_proportions/``

Calculate the class label (i.e. 'Env') of each source using a leave one out
strategy.
``sourcetracker2 gibbs -i otu_table.biom -m map.txt --loo -o source_loo/``

Calculate the proportion of each source in each sink, using 100 burnins.
``sourcetracker2 gibbs -i otu_table.biom -m map.txt -o mixing_proportions/ --burnin 100``

Calculate the proportion of each source in each sink, using a sink 
rarefaction depth of 2500.
``sourcetracker2 gibbs -i otu_table.biom -m map.txt -o mixing_proportions/ --sink_rarefaceiont_depth 2500``

Calculate the proportion of each source in each sink, using ipyparallel to run
in parallel with 8 jobs.
``sourcetracker2 gibbs -i otu_table.biom -m map.txt -o mixing_proportions/ --jobs 8``
```

