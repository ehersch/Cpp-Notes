# Classes

## Concrete types

- Behave like concrete types
- Can be private (only accessible through member functions)

### Arithmetic type

The classic one is complex

```
class complex {
  double re, im; // representation: two doubles
public:
  complex(double r, double i) :re{r}, im{i} {} // constr uct complex from two scalars
  complex(double r) :re{r}, im{0} {} // constr uct complex from one scalar
  complex() :re{0}, im{0} {} // default complex: {0,0}

  double real() const { return re; }
  void real(double d) { re=d; }
  double imag() const { return im; }
  void imag(double d) { im=d; }

  complex& operator+=(complex z) { re+=z.re , im+=z.im; return ∗this; } // add to re and im
  // and return the result

  complex& operator−=(complex z) { re−=z.re , im−=z.im; return ∗this; }

  complex& operator∗=(complex); // defined out-of-class somewhere
  complex& operator/=(complex); // defined out-of-class somewhere
};
```

- `const` indicates these functions do not modify the ojects they are calling

There are also functions that don't need to access the representation of `complex`, so they can be defined separate from the class definition:

```
complex operator+(complex a, complex b) { return a+=b; }
complex operator−(complex a, complex b) { return a−=b; }
complex operator−(complex a) { return {−a.real(), −a.imag()}; } // unary minus
complex operator∗(complex a, complex b) { return a∗=b; }
complex operator/(complex a, complex b) { return a/=b; }
```

### Container

- Object holding a collection of elements (i.e. a Vector is just a container)
- Need a mechanism for collecting and deallocating elements
- Use a destructer to deallocate memory allocated by the constructor

```
class Vector {

private:
  double∗ elem; // elem points to an array of sz doubles
  int sz;

public:
  Vector(int s) : elem{new double[s]}, sz{s} // constructor: acquire resources
  {
    for (int i=0; i!=s; ++i) // initialize elements
      elem[i]=0;
  }
  ~Vector() { delete[] elem; } // destructor: release resources

  double& operator[](int i);
  int size() const;
};
```

The destructor uses the complement operator `~` followed by the class name

The constructor allocates memory on the free store (heap / dynamic memory) using the `new` operator. The destructor cleans this up using `delete`.

> Note, `elem{new double[s]}, sz{s}` is part of the member initialization list. It tells C++ how to construct the data members before the body `{...}` runs.

### Initializing Containers

Need a convenient way to add elements to a container (Vector).

Initialize-list constructor to initialize list of doubles. `push_back()` adds new element to the back of the sequence.

```
class Vector {
public:
  Vector(std::initializ er_list<double>); // initialize with a list of doubles
  // ...
  void push_back(double); // add element at end, increasing the size by one
  // ...
};
```

ex/

```
Vector read(istream& is)
{
  Vector v;
  for (double d; is>>d;) // read floating-point values into d
    v.push_back(d); // add d to v
  return v;
}
```

The `std::initializer_list` is a standard library thing to know how to treat `{1,2,3,4}`. We can write

```
Vector v1 = {1,2,3,4,5}; // v1 has 5 elements
Vector v2 = {1.23, 3.45, 6.7, 8}; // v2 has 4 elements
```

We could define Vector's initialize-list constructor like:

```
Vector::Vector(std::initializ er_list<double> lst) // initialize with a list
  :elem{new double[lst.size()]}, sz{static_cast<int>(lst.size())}
{
  copy(lst.begin(),lst.end(),elem); // copy from lst into elem (§10.6)
}
```

A `static_cast` does not check the value it is converting; the programmer is trusted to use it correctly.

## Abstract Types

`complex` and `Vector` are concrete because their representation is part of their definition. This is different from an _abstract type_ which insulates the user from implementation details.

First define a class `Container` which is more abstract of `Vector`

```
class Container {
public:
  virtual double& operator[](int) = 0; // pure virtual function
  virtual int size() const = 0; // const member function (§4.2.1)
  virtual ~Container() {} // destructor (§4.2.2)
};
```

This is just an interface to containers defined later. `virtual` means "may be redefined later in a class derived from this one."

`=0` says the function is _pure virtual_ (some class derived from `Container` _must_ define the function). Thus, a `Container` is not just a `Container`: it MUST implement `operator[]()` and `size()`.

> A class with a pure virtual function is called an abstract _class_

A `Container` can be used like this:

```
void use(Container& c)
{
  const int sz = c.size();

  for (int i=0; i!=sz; ++i)
    cout << c[i] << '\n';
}
```

A container that implementes the functions required by the interface defined by the abstract class `Container` could use the concrete class `Vector`:

```
class Vector_container : public Container { // Vector_container implements Container

  Vector v;

  public:

    Vector_container(int s) : v(s) { } // Vector of s elements
    ~Vector_container() {}

    double& operator[](int i) { return v[i]; }
    int size() const { return v.size(); }
};
```

`:public` means "is derived from." The members `operator[]()` and `size()` _override_ the corresponding members in the base class `Container`. The destructor `~Vector_container()` overrides the base destructor.

> Note the member destructor `Vector()` is implicitly invoked by its class's destructor `~Vector_container()`

## Virtual Functions

```
void use(Container& c)
{
  const int sz = c.size();

  for (int i=0; i!=sz; ++i)
    cout << c[i] << '\n';
}
```

How is the call `c[i]` and `use()` resolved to the right `operator[]()`? When calling `use()`, `List_container`'s `operator[]()` must be called. Or `Vector_container`'s `operator[]()` is called for other function. This uses a **virtual function table**! Each class with virtual functions has its own `vtbl`.

## Class Hierarchies

- Can override i.e. `void draw() const override;`
- Can copy constructors

> Read in Chapters 4.5 and 4.6!

### Essential operations

```class X {

public:
  X(Sometype); // "ordinary constructor": create an object
  X(); // default constructor
  X(const X&); // copy constructor
  X(X&&); // move constructor
  X& operator = (const X&); // copy assignment: clean up target and copy
  X& operator = (X&&); // move assignment: clean up target and move
  ~X(); // destructor: clean up
};
```

There are five situations in which an object is copied or moved:

- As the source of an assignment
- As an object initializer
- As a function argument
- As a function return value
- As an exception

### Resource Management
