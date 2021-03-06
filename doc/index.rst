faststat
========

faststat is a *streaming*, *light-weight* statistics library designed for embedding
in other Python applications.  *Steaming* means
that faststat operates on data points as they arrive, without needing to store
previous data points.  *Light-weight* means that statistics do not take up a great
deal of CPU or RAM.  Adding a data point to a stat object is a 0.5 - 3 microsecond operation.  
Each stat object takes about 4kiB of memory.

.. toctree::
   :maxdepth: 2

Basic usage
-----------
The most basic usage of faststat is to create a Stats object to represent some continuous
variable, and then add data points to it.  

::

   >>> import faststat
   >>> s = faststat.Stats()
   >>> for i in range(100):
   ...    s.add(i)
   ...
   >>> s
   <faststat.Stats n=100 mean=49.5 quartiles=(23.4, 49.6, 73.8)>

Collected data
--------------
The following data is collected for each point.  A data point is considered to be `(x, t)`
where `x` is a floating point value, and `t` is the system clock at the time the data was passed
to faststat.

=============== ====================================================================================
attribute       description
=============== ====================================================================================
n               The number of data points.

mean            The arithmetic mean, also known as expected value E(x) or 
                :math:`\bar{x}`. Defined as :math:`\bar{x} = \frac{x_1 + x_2 + ... + x_n}{n}`

max             The largest value seen.

maxtime         The time of the largest data point.

min             The smallest value seen.

mintime         The time of the smallest data point.

lasttime        The time of the most recent data point.

percentiles     A dictionary of approximate percentiles.

buckets         Counts of data points which have occurred in different ranges.
                Essentially logarithmic-scale histogram data.      

variance        The variance.  See http://en.wikipedia.org/wiki/Variance

skewness        The skewness.  See http://en.wikipedia.org/wiki/Skewness

kurtosis        The kurtosis.  See http://en.wikipedia.org/wiki/Kurtosis

geometric_mean  The geometric mean, or NaN if any points are <= 0.  
                See http://en.wikipedia.org/wiki/Geometric_mean

harmonic_mean   The harmonic mean, or NaN if any points are <= 0.
                See http://en.wikipedia.org/wiki/Harmonic_mean

expo_avgs       A dictionary mapping exponential decay factors to current values.
                See http://en.wikipedia.org/wiki/Moving_average#Exponential_moving_average

get_prev()      The most recent data points.

get_topN()      The largest data points

window_avg      The mean of the data points in get_prev().
=============== ====================================================================================


Error
-----

There is no proven error bound on the algorithm used to calculate percentiles.  However, empirically
the error is observed to be low.  It is worth noting that the percentile algorithm used (P2) performs
interpolation of values.  Therefore for a sequence consisting of ~50% 1's and ~50% 2's, the algorithm
would report a median around 1.5.


Examples
--------

Tracking the average number of items present each time a new item was added.

.. code-block:: python

   import collections
   import faststat

   class Queue(object):
      def __init__(self):
         self.deq = collections.deque()
         self.put_stats = faststat.Stats()

      def put(self, item):
         self.put_stats.add(len(self.deq))
         self.deq.append(item)

      def get(self):
         return self.deq.popleft()

