# Modularity

Declarations vs interfaces:

- A declaration specifies all that's needed to use a function or type
  - The function bodies/definitions are elsewhere

## Separate Compilation

Header file: `Vector.h `

and to access this interface we incude it

`#include "Vector.h"`

Implement this in `Vector.cpp`

```
// Vector.cpp:

#include "Vector.h" // get the interface

Vector::Vector(int s)
  :elem{new double[s]}, sz{s} // initialize members
{
}

double& Vector::operator[](int i)
{
  return elem[i];
}

int Vector::size()
{
  return sz;
}
```

## Namespaces

Mechanism for expressing that some declarations belong together and that their names shouldn't clash with other names. i.e. a complex number type:

```
namespace My_code {
  class complex {
    // ...
  };
  complex sqr t(complex);
  // ...
  int main();
}

int My_code::main()
{
  complex z {1,2};
  auto z2 = sqrt(z);
  std::cout << '{' << z2.real() << ',' << z2.imag() << "}\n";
  // ...
};

int main()
{
  return My_code::main();
}

```

By putting code into `My_code`, make sure names do not conflict with standard library names in namespace `std`. This is smart because the standard library has support for `complex` arithmetic.

Access via: `std::cout` or `My_code::main`

Gain access with a using-directive: `using namespace std`

- This allows us to just write `cout` instead of `std::cout`

## Error Handling

### Exceptions

Say a vector attempts to access an out-of-range item... throw `out_of_range` exception:

```
double& Vector::operator[](int i)
{
  if (i<0 || size()<=i)
    throw out_of_range{"Vector::operator[]"};
  return elem[i];
}
```

`throw` transfers control to the hendler of this type

Try blocks!

```
void f(Vector& v)
{
  // ...
  try { // exceptions here are handled by the handler defined below
    v[v.size()] = 7; // try to access beyond the end of v
  }
  catch (out_of_rang e) { // oops: out_of_range error
    // ... handle range error ...
  }
  // ...
}
```

A function that should never throw an exception can be declared `noexcept`:

```
void user(int sz) noexcept
{
  Vector v(sz);
  iota(&v[0],&v[sz],1); // fill v with 1,2,3,4...
  // ...
}
```

If the `user()` still throws, the standard library function `terminate()` is called to immediately terminate the program.

### Invariants

```
Vector::Vector(int s)
{
  if (s<0)
    throw length_error{};
  elem = new double[s];
  sz = s;
}
```

Then

```
void test()
{
  try {
    Vector v(−27);
  }
  catch (std::length_error) {
  / / handle negative size
  }
  catch (std::bad_alloc) {
    // handle memory exhaustion
  }
}
```

Can throw in: `std::terminate();`

Recall, in main function (`int main`), `return 0` is successful and `return 1` is error.

### Static Assertations

Exceptions report errors found at run time. We want to find errors at compile time! Perform simple checks for compile-time messages:

`static_assert(4<=sizeof(int), "integers are too small"); // check integer size`

`static_assert(A,S)` prints `S` as a compiler error message if `A` is not `true`. This is used when we want to make assertions about types used as parameters. Runtime-checked assertions are `exceptions`.
