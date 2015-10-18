# efmaral
Efficient Markov Chain word alignment

`efmaral` is a tool for performing word alignment using a Gibbs sampling for
a Bayesian extension of the IBM alignment models.

The model used is described in my thesis, which you may want to consider
citing if you use this tool:
* [Robert Östling](http://www.robos.org/). [Bayesian Models for Multilingual Word Alignment](http://urn.kb.se/resolve?urn=urn:nbn:se:su:diva-115541). PhD thesis, Stockholm University, 2015.

It uses the same input and output formats as
[fast_align](https://github.com/clab/fast_align), for which it can be used as
a (faster and more accurate) plug-in replacement.

## Installing

`efmaral` is implemented in Python/C and requires the following software to be
installed:

 * Python 3 (tested with version 3.4)
 * gcc (tested with version 4.9) and GNU Make
 * Cython (tested with version 0.21)
 * NumPY (tested with version 1.8.2)

These can be installed on Debian 8 (jessie) using the following command as root:

    apt-get install python3-dev python3-setuptools cython3 gcc make python3-numpy

Then, clone this repository and run `make`:

    git clone https://github.com/robertostling/efmaral.git
    cd efmaral
    make

If everything works, you can try directly to evaluate with a small part of
the English-Hindi data set from WPT-05:

    python3 scripts/evaluate.py efmaral \
        3rdparty/data/test.eng.hin.wa \
        3rdparty/data/test.eng 3rdparty/data/test.hin \
        3rdparty/data/trial.eng 3rdparty/data/trial.hin

This uses the small trial set as training data, so the actual figures are
poor.

## Usage

The default values of `efmaral` should give acceptable results, but for a full
list of options, run:

    ./efmaral.py --help

Given a parallel text in `fast_align` format, `efmaral` can be used in the same
way as `fast_align`. First, we can convert some of the WPT-05 data above to
the `fast_align` format:

    python3 scripts/wpt2fastalign.py \
        3rdparty/data/test.eng 3rdparty/data/test.hin >test.fa

Then, we can run `efmaral` on this file:

   ./efmaral.py -i test.fa >test-fwd.moses 

If we want symmetrized alignments, also run in the reverse direction, then run
atools (which comes with `fast_align`) to symmetrize:

    ./efmaral.py -r -i test.fa >test-back.moses
    atools -i test-fwd.moses -j test-back.moses -c grow-diag-final-and >test.moses

`efmaral` also supports reading Europarl-style inputs directly, such as the
WPT data, by providing two filename arguments to the `-i` option:

    /efmaral.py -i 3rdparty/data/test.eng 3rdparty/data/test.hin >test-fwd.moses
    ...

There is a convenience script to perform both alignments + symmetrization:

    scripts/align_symmetrize.sh 3rdparty/data/test.eng 3rdparty/data/test.hin test.moses

## Tips and tricks

The three most important options are probably:

 * `--length`: number of iterations relative to the number determined
   automatically (based on the number of sentences). The default value of 1.0
   means no change, whereas e.g. 0.2 would result in a 5x speedup (unless one
   hits the bottom number of 4 or top value of 250 iterations) -- probably at
   the cost of some accuracy.
 * `--null-prior`: prior probability of NULL alignment, this can to some
   extent be used to trade recall for precision, by setting a higher value
   than the default 0.2.
 * `--samplers`: number of independent samplers to use. This reduces
   initialization bias, and is the only way for `efmaral` to utilize multiple
   CPU cores since each is run in a separate thread. The default is 2, but for
   maximum speed use a value of 1. A value much higher than 4 is probably not
   very useful.
