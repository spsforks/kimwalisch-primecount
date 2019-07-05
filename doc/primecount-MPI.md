# primecount MPI

[![Build Status](https://travis-ci.org/kimwalisch/primecount.svg)](https://travis-ci.org/kimwalisch/primecount)
[![Build Status](https://ci.appveyor.com/api/projects/status/github/kimwalisch/primecount?branch=master&svg=true)](https://ci.appveyor.com/project/kimwalisch/primecount)
[![Github Releases](https://img.shields.io/github/release/kimwalisch/primecount.svg)](https://github.com/kimwalisch/primecount/releases)

This is the distributed version of primecount which uses the
[MPI](https://en.wikipedia.org/wiki/Message_Passing_Interface) library
for inter-process communication and which automatically distributes
the computation onto cluster nodes. primecount MPI scales nearly
linearly (up to 96.8% efficiency) for large input values and has been
tested successfully with clusters of up to 50 nodes.

Breaking the next world record pi(10<sup>28</sup>) would take about 6
years on the fastest shared memory server currently available
(Intel Xeon, 36 CPU cores). Hence it has become necessary to go
distributed!

## Build instructions (Unix-like OSes)

First install the prerequisites:
```sh
sudo apt-get install g++ make cmake libopenmpi-dev openmpi-bin
```

Then build primecount MPI using:
```sh
cmake -DWITH_MPI=ON .
make -j
```

## Usage example

```sh
# Distribute pi(10^23) computation onto 30 cluster nodes
mpiexec -n 30 --map-by node ./primecount 1e23 --status
```

```--map-by node``` ensures that only one primecount process will be
created on each cluster node. This is important for performance as
primecount will by default use all available CPU cores on each cluster
node using [OpenMP](https://en.wikipedia.org/wiki/OpenMP)
multi&#8209;threading.

## Performance tips

* You should create only one process per cluster node as primecount MPI
  will by default use all available CPU cores on each cluster node using
  [OpenMP](https://en.wikipedia.org/wiki/OpenMP) multi-threading.
* Be careful when submitting multiple future primecount MPI jobs, they
  might be run simultaneously on the same hardware instead of one after
  the other. This will of course deteriorate performance.
* Ideally all cluster nodes should have an equal number of CPU cores
  because primecount MPI evenly (statically) distributes the work among
  the cluster nodes.
* If possible you should
  [enable transparent huge pages](https://github.com/kimwalisch/primecount#performance-tips)
  on all cluster nodes in order to reduce [TLB (translation lookaside buffer)](https://en.wikipedia.org/wiki/Translation_lookaside_buffer)
  cache misses. This will usually provide a minor speedup.

## Benchmark pi(10<sup>23</sup>)

<table>
  <tr align="center">
    <td><b>Cluster nodes</b></td>
    <td><b>CPU cores</b></td>
    <td><b>Seconds</b></td>
    <td><b>Speedup</b></td>
    <td><b>Efficiency</b></td>
  </tr>
  <tr align="right">
    <td>1</td>
    <td>16</td>
    <td>30,807.46</td>
    <td>1.00 x</td>
    <td>100.00%</td>
  </tr>
  </tr>
  <tr align="right">
    <td>5</td>
    <td>80</td>
    <td>6,645.98</td>
    <td>4.63 x</td>
    <td>92.60%</td>
  </tr>
  </tr>
  <tr align="right">
    <td>10</td>
    <td>160</td>
    <td>3,204.08</td>
    <td>9.61 x</td>
    <td>96.10%</td>
  </tr>
  </tr>
  <tr align="right">
    <td>20</td>
    <td>320</td>
    <td>1,590.61</td>
    <td>19.36 x</td>
    <td>96.80%</td>
  </tr>
  </tr>
  <tr align="right">
    <td>30</td>
    <td>480</td>
    <td>1,064.26</td>
    <td>28.94 x</td>
    <td>96.46%</td>
  </tr>
  <tr align="right">
    <td>40</td>
    <td>640</td>
    <td>827.73</td>
    <td>37.21 x</td>
    <td>93.02%</td>
  </tr>
  <tr align="right">
    <td>50</td>
    <td>800</td>
    <td>718.15</td>
    <td>42.89 x</td>
    <td>85.78%</td>
  </tr>
</table>

The pi(10<sup>23</sup>) benchmark above was run on an
[EC2 cluster](https://aws.amazon.com/ec2/) where each cluster node had
2 CPUs of type Intel Xeon E5-2680 v2 (2.80GHz, 8 CPU cores, 16 threads).
The efficiency drops beyond 40 cluster nodes because the input
10<sup>23</sup> is too small for such a large number of nodes.
For 10<sup>24</sup> and 50 cluster nodes the efficiency is 93,65%.

## Command-line options

```
Usage: primecount x [OPTION]...
Count the primes below x <= 10^31 using fast implementations of the
combinatorial prime counting function.

Options:

  -d,    --deleglise_rivat  Count primes using Deleglise-Rivat algorithm
         --legendre         Count primes using Legendre's formula
         --lehmer           Count primes using Lehmer's formula
  -l,    --lmo              Count primes using Lagarias-Miller-Odlyzko
  -m,    --meissel          Count primes using Meissel's formula
         --Li               Approximate pi(x) using the logarithmic integral
         --Li_inverse       Approximate nth prime using Li^-1(x)
  -n,    --nthprime         Calculate the nth prime
  -p,    --primesieve       Count primes using the sieve of Eratosthenes
         --phi=<a>          phi(x, a) counts the numbers <= x that are
                            not divisible by any of the first a primes
         --Ri               Approximate pi(x) using Riemann R
         --Ri_inverse       Approximate nth prime using Ri^-1(x)
  -s[N], --status[=N]       Show computation progress 1%, 2%, 3%, ...
                            [N] digits after decimal point e.g. N=1, 99.9%
         --test             Run various correctness tests and exit
         --time             Print the time elapsed in seconds
  -t<N>, --threads=<N>      Set the number of threads, 1 <= N <= CPU cores
  -v,    --version          Print version and license information
  -h,    --help             Print this help menu

Advanced Deleglise-Rivat options:

  -a<N>, --alpha=<N>        Tuning factor, 1 <= alpha <= x^(1/6)
         --P2               Only compute the 2nd partial sieve function
         --S1               Only compute the ordinary leaves
         --S2_trivial       Only compute the trivial special leaves
         --S2_easy          Only compute the easy special leaves
         --S2_hard          Only compute the hard special leaves
```