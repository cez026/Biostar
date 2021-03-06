{% load pytags %}

Although genomes exist in three-dimensional space, in bioinformatics we often simplify our
model of the genome to two dimensions in order to facilitate visualization and analysis
tasks. Hence, coordinates for spatially describing genomic loci tend to follow the general
pattern of chromosome:start-end where "start" and "end" are both integers that represent the
left and right positions of an interval if we think of the origin always being the furthest
left point. Since nucleic acid molecules are polar we can add strand information in order to
understand whether left or right indicates the 5' or 3' end of an interval.

A common format used to represent an interval in genomics is called the **BED format** and
it looks something like this:

chr1  50  100 int_1 50  +

Here the first column represents a sequence ID, in this case a chromosome, the second column
indicates the start of the interval, the third column the end of the interval, the fourth
column a name we can give our interval, the fifth a score we can assign to our interval (in
this case it's simply the width of the interval), and the sixth column tells us where the 5'
and 3' ends of the interval are. Since this interval exists on the positive strand we know
that the second column (50) represents the coordinate for the 5' end of the interval while
the third column represents the coordinate of the 3' interval. If the sixth column were "-",
then we would know that the third column (100) represents the 5' end of the interval while
the second column represents the 3' end. The width is the same either way.

This relatively simple format can represent an relatively diverse amount of data. For
example, one could imagine genomic mappings in this format, telling us where certain reads
mapped onto a reference sequence. Now imagine those reads originated from transcribed genes
and we wanted to know exactly how many reads map to a region where there's also a gene?

Turns out we can also represent a gene in this format by giving it coordinates and
calculating whether those coordinates overlap one another. Consider the following gene:

chr1  25  300 gene_1  275 +

We can see that this gene is on the same chromosome and the same strand as our interval
example from earlier. We can also see just by looking that an interval starting from
nucleotide 50 and ending on nucleotide 100 falls into the large interval of our gene. Thus
we would say that int_1 intersects gene_1 fully. That is all of int_1 falls into gene_1 -
there are no overhangs on either side.

Now imagine we have hundreds of millions of reads and tens of thousands of genes and we want
to count the number of reads that overlap with every gene. It turns out there are a lot of
different ways of doing this and that it can get computationally intensive once you scale
up.

The bedtools suite allows us to perform various operations on the same kinds of intervals
introduced above in a computationally efficient manner.

Below are three of the many tools offered in the bedtools suite that can answer a vast array
of biologically relevant questions. Following these three we'll show you how to combine
various tools into a pipeline to answer even more complicated questions.

- - -

#### Which of my intervals overlap?

A common application of looking for the intersection between two intervals is in assigning
short read mappings to a gene based on its mapping coordinates overlapping the coordinates
of said gene like in our example above. Here we'll look to overlap a set of intervals
representing reads with a set of intervals representing two genes.

{% code "intervals/bedtools-intersect.sh" %}

[Documentation](http://bedtools.readthedocs.org/en/latest/content/tools/intersect.html)


- - -

#### Which of my intervals are closest?

What if we're not interested in looking at the intersection between two sets of intervals
but instead which intervals are nearby and how close they are? We can use the bedtools
closest command to help with that. Here we'll use it to determine the closest downstream
genes to a set of intervals representing identified ChIP-seq peaks.

{% code "intervals/bedtools-closest.sh" %}

[Documentation](http://bedtools.readthedocs.org/en/latest/content/tools/closest.html)


- - -
#### How similar are two sets of intervals?

Now imagine we have two sets of intervals from two different analyses and we want to
calculate how similar they are. We could just use the bedtools intersect command to see
whether there are any intervals that don't intersect with each other but that doesn't tell
us exactly *how much* intersection there is between the two sets. The jaccard command can
calculate a statistic for us that quantifies how similar two sets of intervals are to one
another. Here we'll use it to compare a set of annotations derived from an assembly to a set
of curated reference annotations.

{% code "intervals/bedtools-jaccard.sh" %}

[Documentation](http://bedtools.readthedocs.org/en/latest/content/tools/jaccard.html)
---


#### Using multiple tools to create a pipeline

One of the best parts about the bedtools suite is the ability to combine various commands
into a pipeline with relative ease. Consider the following scenario:

We have a file that represents all genes and another file that represents some genes we're
not interested in. We want to count the number of intersections of a set of reads we have
but only for the genes we're interested in because we plan to apply a normalization factor
and don't want to include certain genes in the calculation of that scaling factor.

Could we do this with bedtools? Yes! Yes you can.

{% code "intervals/bedtools-pipeline.sh" %}
