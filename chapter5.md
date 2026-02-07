# Templates

- A class or function we parameterize with a set of types or values (like vector of int, double, etc)

## Parameterized Types

Generalize vector-of-doubles to a vector-of-anything type making it a template and replacing the specific type `double` with a parameter:

```
template<typename T>

class Vector {
private:
  T* elem; // elem points to an array of sz elements of type T
  int sz;

public:
  explicit Vector(int s); // constr uctor: establish invariant, acquire resources
  ~Vector() { delete[] elem; } // destr uctor: release resources

  // ... copy and move operations ...

  T& operator[](int i);
  const T& operator[](int i) const;
  int size() const { return sz; }
};
```

`template<typename T>` prefix makes `T` a parameter of the declaration it prefixes.

Another way of doing this is

```
template<typename T>

Vector<T>::Vector(int s)
{
  if (s<0)
    throw Negative_size{};
  elem = new T[s];
  sz = s;
}

template<typename T>
const T& Vector<T>::operator[](int i) const
{
  if (i<0 || size()<=i)
    throw out_of_range{"Vector::operator[]"};
  return elem[i];
}
```

Can define vector like `Vector<char> vc(200);`

The `>>` in `Vector<list<int>>` terminates the nested arguments for `Vector<list<int>> vli(45);`

Use a vector like this:

```
void write(const Vector<string>& vs) // Vector of some strings
{
  for (int i = 0; i!=vs.size(); ++i)
    cout << vs[i] << '\n';
}
```

To support range-for loop for vector, define `begin()` and `end()`

```
template<typename T>
T∗ begin(Vector<T>& x)
{
  return x.size() ? &x[0] : nullptr; // pointer to first element or nullptr
}

template<typename T>
T∗ end(Vector<T>& x)
{
  return begin(x)+x.size(); // pointer to one-past-last element
}
```

Now we can do

```
void f2(Vector<string>& vs) // Vector of some strings
{
  for (auto& s : vs)
    cout << s << '\n';
}
```

## Function Templates

Can use templates to parameterize types and algorithms in the standard library:

```
template<typename Container, typename Value>
Value sum(const Container& c, Value v)
{
  for (auto x : c)
    v+=x;
  return v;
}
```

`Value` argument and the function argument `v` allow the caller to specify the type and initial value of the accumulator.

## Function Objects

Also called a _functor_, used to define objects that can be called like functions:

```
template<typename T>
class Less_than {
  const T val; // value to compare against
public:
  Less_than(const T& v) :val(v) { }
  bool operator()(const T& x) const { return x<val; } // call operator
};
```

The function `operator()` implements "function call," "call," or "application" operator ().

> "A functor is pretty much just a class which defines the operator(). That lets you create objects which "look like" a function:" -- https://stackoverflow.com/questions/356950/what-are-c-functors-and-their-uses'

## Variadic Templates

Template can accept an arbitrary number of arguments of arbitrary types:

```
void f() { }

template<typename T, typename ... Tail>

void f(T head, Tail... tail)
{
  g(head); // do something to head
  f(tail...); // try again with tail
}
```

## Aliases

Might want a synobym for a type or template.

`using size_t = unsigned int;`

The actual type named `size_t` is implementation-dependent, so another implementation `size_t` may be an `unsigned long`. The alias `size_t` allows for portable code.

It is very common for a parameterized type to provide an alias for types related to their template arguments. For example:

```
template<typename T>
class Vector {
public:
  using value_type = T;
  // ...
}
```
