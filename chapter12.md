# Numerics

## Mathematical Functions

`<cmath>` has a lot of functions like `abs(x)`, `floor(x)`, etc.

Versions for complex functions are in `<complex>`

## Numerical Algorithms

`<numeric>` has generalized numerical algorithms, like `accumulate(b,e,i) // x is the sum of i and elements of [b:e)`

`inner_product(b,e,b2,i) // inner product of [b:e) an [b2:b2+(e-b))`

## Complex Numbers

To support `float`, `double`, etc. the `complex` library is a template. The usual arithmetic operations are supported:

`complex<long double> ld {fl + sqrt(db)};`

`sqrt()` and `pow()` are among the usual functions defined in `<complex>`

## Random Numbers

`<random>` standard library---a RNG consists of two parts:

- an _engine_ produces a sequence of pseudo-random values
- a _distribution_ that maps those balies into a mathematical distribution in a range

Example: `uniform_int_distribution`, `normal_distribution`, `exponential_distribution`

```
using my_engine = default_random_engine; // type of engine
using my_distribution = uniform_int_distribution<>; // type of distribution

my_engine re {}; // the default engine
my_distribution one_to_six {1,6}; // distribution that maps to the ints 1..6
auto die = bind(one_to_six,re); // make a generator

int x = die(); // roll the die: x becomes a value in [1:6]
```

`bind()` makes a function object that will invoke its first argument given its second.

## Vector Arithmetic

Just used to hold values, so DOES NOT include arithmetic in `vector`. The standard library `<valarray>` is vector-like and more amenable to optimization for numerical computation:

```
valarray<double> a = a1*3.14 + a2/a1;
double d = a2[7];
```

## Numeric Limits

In `<limits>` the standard library provides classes that describe properties of built-in types (like max exponent of a `float` or the number of bytes in an `int`):

```
static_asser t(numeric_limits<char>::is_signed,"unsigned characters!");

static_assert(100000<numeric_limits<int>::max(),"small ints!");
```

Note, `numeric_limits<int>::max()` is a `constexpr` function.
