# PrimeSieve

[![Build Status](https://travis-ci.org/jlapeyre/PrimeSieve.jl.svg?branch=master)](https://travis-ci.org/jlapeyre/PrimeSieve.jl)

This package provides an interface to tables of primes and a sieve
library.  It is extremely fast, in particular the prime pi function,
which combines sieving with table lookup. For instance, a particular Intel i7
machine gives π(x) for any x<10^13 in the worst case in about
150ms.

```julia
julia> @time countprimes("10^13-1")
elapsed time: 0.15288837 seconds (1312 bytes allocated)
346065536839
```

There are multi-threaded and single-threaded versions of most
functions.

See LICENSE.md for links to the authors of the tables and the library. All
the credit for the utility of this package goes to them.

I am unaware of binaries of libprimesieve for Windows and OSX, so these
are not installed automatically.

### Tables

http://www.ieeta.pt/~tos/primes.html

### Primesieve

http://primesieve.org/

### Data types

There is a risk of overflow when constructing and giving arguments to
functions in this package. The easiest way to avoid this is to put arguments in
quotes: eg ```countprimes("10^19", "10^19+100")```.

### Example

```julia
julia> using PrimeSieve
julia> @time countprimes(10^17 + 10^14 + 10^10)
elapsed time: 3.729049749 seconds (168 bytes allocated)
2626112053757377
```

To see what happened, we can look in the tables:

```julia
julia> primelookup(10^17 + 10^14 + 10^10)
(14,(2626111798288135,100100000000000000,10000000000))
```

The 14th table was used. The value of prime pi for ```10^17+10^14```,
```2626111798288135``` is in the table, and the primes in an
interval of length ```10^10``` must be found with the sieves.

See the description of ```primelookup``` below.

## Functions

### countprimes

Count the number of primes (or prime tuplets) in an interval. This
looks up the largest value in the table that is smaller than the
requested value and computes the remaining values. This is the only
function in the package that uses tables.

Usage:
```julia
countprimes(stop)            # count the number of primes less than or equal to stop
countprimes(start,stop)      # count the number of primes >= start and <= stop
ntcountprimes([start],stop)  # Do not use table lookup, only sieving
countprimes([start], stop, tuplet=n) # Count prime n-tuplets
```

The default value of start is 1.
The optional keyword argument 'tuplet' may take values between 1 and 6, that is
primes, through prime sextuplets. Tables are implemented only for
'tuplet' equal to one, although Oliveira e Silva has compiled tables for
twin primes.

Examples
```julia
countprimes(100)  # the number of primes x such that  1 <= x <= 100
25
countprimes(100,tuplet=3)  # the number of prime triplets between 1 and 100
8
countprimes(10,1000,tuplet=6)  # the number of prime sextuplets between 100 and 1000
1     
```

If you quote the arguments (either as an expression or a string),
they will be converted to Int128. This prevents overflow.
```
countprimes("10^19+10^9")
234057667299198865
```

### genprimes

Return an array of all primes ```>= start``` and ```<= stop```

Usage
```julia
genprimes([start=1],stop)
```

### nprimes

Return an array of the first ```n``` primes ```>= start```.

Usage
```julia
nprimes(n,[start=1])
```
### single threaded versions

Prepending 's' to the function name of any of the above functions
calls a single-threaded version. There is no routine ```sntcountprimes```
and ```scountprimes``` does not use tables.

Usage
```julia
scountprimes([start],stop, tuplets=1)
```

### printprimes

Print all primes (or prime n-tuplets) that are ```>= start``` and ```<= stop```

Usage
```julia
printprimes([start],stop, [tuplet=1])
```

The default value of 'start' is 1.
The optional keyword argument 'tuplet' may take values between 1 and 6.

### primesievesize

Get, set the sieve size in kilobytes. (setting does not seem to work)
```sz``` must satisfy  ```1 <= sz <= 2048```

Usage
```julia
primesievesize()
primesievesize(sz)
```

### primenumthreads

Get, set the number of threads used in parallel routines. By default, the
OMP default is used.

Usage
```julia
primenumthreads()
primenumthreads(numthreads)
```

### primemaxstop

Return the largest value (as a ```Uint64```) that can be passed as the parameter
stop.

Usage
```julia
primemaxstop()
```

### primetest

Run a test of the algorithms

Usage
```julia
primetest()
```

## Tables of prime pi function

### primetableinfo

Print information about the prime pi tables.

```julia
julia> primetableinfo()
Tables of π(x). Listed are: table number, increment in x (and first value of x),
number of entries in the table, largest x in table.

table  incr    tab len  max x
1      10^1    10^4     10^5
2      10^2    10^4     10^6
3      10^3    10^4     10^7
4      10^4    10^4     10^8
5      10^5    10^4     10^9
6      10^6    10^4     10^10
7      10^7    10^4     10^11
8      10^8    10^4     10^12
9      10^9    10^4     10^13
10     10^10   10^4     10^14
11     10^11   10^4     10^15
12     10^12   10^4     10^16
13     10^13   10^4     10^17
14     10^14   10^4     10^18
15     10^15   10^4     10^19
16     10^16   10^4     10^20
17     10^17   10^3     10^20
18     10^18   10^2     10^20
19     10^19   10^2     10^21
20     10^20   10^2     10^22
21     10^21   10^2     10^23
22     10^22   10^1     10^23
```

### primelookup

Look up a value of the prime pi function in the tables. This is only provided
to aid in understanding the behavior of ```countprimes```.

Usage
```julia
primelookup(x)
```

A tuple of a single element and another tuple of three elements is returned:

```julia
(j,(p,y,rem))
```
* ```j``` is the number of the best table found
* ```y``` is the largest index satisfying ```y<x``` found.
* ```p``` is the value of prime pi at ```y```
* ```rem``` is ```x-y```

### primetables

The array of type ```Array{PrimeTable,1}``` containing the prime tables.
See tables.jl for the format.

Example
```julia
show(map(length,primetables)) # see the number of tables and their lengths
```

### primetablefilename

Function returning the path to the file containing the prime pi tables.
The tables are loaded when the package is loaded.

## Other details

For ```x>typemax(Int)```, you need to explicitly ask for a bigger data type.
For instance,

```julia
julia> countprimes(int128(10)^23)
1925320391606803968923
```

This example returned a value from the table.
The argument was larger than than primemaxstop().


With any of the routines, you can quote the arguments and they will be converted
to the appropriate type.

```julia
julia> countprimes(:(10^23))
1925320391606803968923
julia> countprimes("10^19 + 10^9")
234057667299198865
```

Routines that use the tables will convert the arguments to Int128. This is because
some indices in the tables are greater than ```typemax(Uint64)```.  Routines that
only use the sieve will be converted to ```Uint64```, which is the data type that
the sieve routines use.


The largest stop value that may be given is ```2^64 - 10 * 2^32```.
The largest start value that may be given is ```2^64 - 11 * 2^32```.
The sieve works with the ```Uint64``` data type. But conversions are done depending
on the types of start, stop, and n.

```countprimes``` returns ```Int128```, because it uses tables and sieves
The other routines only support smaller data types.

### primetabletype()

Return data type of tables. This should be Int128. The largest values cannot
be used together with the sieve.

### primesievetype()

Return the native prime sieve type. This should be Uint64. libprimesieve
returns the data in various integer formats. These are chosen by the Julia
interface by the type of the ```start``` parameter.

### eltype(t::PrimeTable)

Return element type of values in table

## Bugs

Interrupting a call to the sieves usually does not cause a memory error.
But, libprimesieve apparently has some static state, such that,
after the interrupt, subsequent sieving runs much slower, and may not
give correct results.

<!--  LocalWords:  PrimeSieve lookup multi md libprimesieve OSX julia
 -->
<!--  LocalWords:  Primesieve countprimes primelookup th tuplets sz
 -->
<!--  LocalWords:  ntcountprimes tuplet genprimes nprimes Prepending
 -->
<!--  LocalWords:  sntcountprimes scountprimes printprimes tuple OMP
 -->
<!--  LocalWords:  primesievesize primenumthreads numthreads Uint jl
 -->
<!--  LocalWords:  primemaxstop primetest primetables PrimeTable incr
 -->
LocalWords:  Oliveira primetableinfo len primetablefilename eltype eg
LocalWords:  primetabletype primesievetype typemax
