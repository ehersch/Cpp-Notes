# Utilities

## Resource Management

Need to release data or else may get memory leaks!

> For example, think constructor/destructor pairs

Used for standard-library lock classes:

```
mutex m; // use to protect access to shared data

void f()
{
  unique_lock<mutex> lck {m}; // acquirethe mutex m

  // manipulate shared data
}
```

Thread will not procede until `lck`'s constructor has aquired `mutex, m`

## `unique_ptr` and `shared_ptr`

What do we do with objects allocated on free store? In `<memory>` the standard library provides two "small pointers" to help manage objects on the free store:

`unique_ptr` to represent unique ownership

`shared_ptr` to represent shared ownership

Can use them to prevent memory leaks caused by careless programming:

```
void f(int i, int j) // X* vs. unique_ptr<X>
{
  X∗ p = new X; // allocate a new X
  unique_ptr<X> sp {new X}; // allocate a new X and give its pointer to unique_ptr
  // ...
  if (i<99) throw Z{}; // may throw an exception
  if (j<77) return; // may retur n "ear ly"
  // ...
  p−>do_something(); // may throw an exception
  sp−>do_something(); // may throw an exception
  // ...
  delete p; // destroy *p
}
```

(check TB)

> The idea is want to minimize `new`. `new` allocated data on the heap. Not using `new` will allocate data on the stack (not on free storage).

Example:

`auto p = new Object(s);`

- Here, `p` is a pointer `Object*` and `Object(s)` is allocated on the heap
- Would require `delete`, manual lifetime

`auto p = Object(s);`

- `p` is an object on the stack
- Lifetime is tied to scope, don't need `delete`

The best modern C++ practice:

`auto c = std::make_unique<Object>(s);  // heap, safe ownership`

- Allocated an `Object` on the heap (internally calls `new Object(s)`)
- Wraps the pointer in a `std::unique_ptr<Object>`, so `c` is not an `Object*`... `c` is a smart pointer that _owns_ the object

> Beneficial because this guarantees automatic cleanup. When `c` goes out of scope, `delete object;`

Because this is a **unique** pointer, this guarantees we cannot copy this object:

`auto c2 = c;   // ❌ compile error`

but it can be moved

`auto c2 = std::move(c); // ownership transfers`

> After this move, `c==nullptr` and `c2` owns the object

The whole idea is that the object cannot be shared so this makes for cleanup and lifetime management

## Specialized Containers

`T[n]` is a built-in, fixed-size allocation of `N` elements of type `T`, converts to `T*`

`array<T,N>` like built-in array but problems fixed

`bitset<N>` a fixed sequence of `N` bits

`vector<bool>` sequence of bits compactly stored in a specialization of `vector`

`pair<T,U>` two elements of types `T` and `U`

`tuple<T...>` sequence of arbitrary number of elements of arbitrary types

`basic_string<C>` sequence of characters of type `C`; provides string operations

`valarray<T>` array of numeric values of type `T`, providing numeric operations

### Array

Array can be allocated in stack, object, or static storage. No overhead, without surprising conversions to pointer types

`array<int,3> a1 = {1,2,3}; // element count not optional`

Length must also be CONSTANT. The following does not work:

```
void f(int n)
{
  array<string,n> aa = {"John's", "Queens' "}; // error : size not a constant expression
//
}
```

If need variable element count, use `vector`.

> Why use `array` if `vector` is so flexible? It's simpler! Some speedups, don't have to worry about de-allocation

> Why use `array` over built-in? Disaster that can happen with conversion to pointer.

### Bitset

Can initialize with integer or string

```
bitset<9> bs1 {"110001111"};
bitset<9> bs2 {399}
```

Can also use bitwise operations (`<<` `>>`)

```
bitset<9> bs3 = ~bs1; // complement: bs3=="001110000"
bitset<9> bs4 = bs1&bs3; // all zeros
bitset<9> bs5 = bs1<<2; // shift left: bs5 = "111000000"
```

### Pair and Tuple

`pair` provides `=`, `==`, `<` and such. `make_pair()` makes it easy to create a `pair` without explicitly mentioning type:

`auto pp = make_pair(v.begin(),2)`

`pp.first // pp[0]`
`pp.second // pp[1]`

If you need more than 2 elements, use `tuple`:

`tuple<string,int,double> t2{"Sild",123, 3.14}; // the type is explicitly specified`

`auto t = make_tuple(string{"Herring"},10, 1.23); // the type is deduced to tuple<string,int,double>`

The elements of `tuple` are numbered, rather than named the way elements of `pair` are (`first` and `second`). GTo get compile-time selection of elements, use `get<1>(t)` rather than `get(t,1)` or `t[1]`

## Time

```
using namespace std::chrono;
auto t0 = high_resolution_clock::now();
do_work();
auto t1 = high_resolution_clock::now();
cout << duration_cast<milliseconds>(t1−t0).count() << "msec\n";
```

## Function Adaptors

Takes functin as argument and returns function object which can invoke the original. `bind()` and `mem_fn()` adaptors do argument binding (_currying_ or _partial evaluation_)

### `bind()`

Produces a function object that can be called with the remaining arguments:

```
double cube(double);

auto cune2 = bind(cube,2);
```

A call `cube2()` will invoke `cube` with argument 2.

```
using namespace placeholders;

void f(int, const string&);
auto g = bind(f,2,_1); // bind f()'s first argument to 2
f(2, "hello");
g("hello"); // also calls f(2, "hello")
```

The `_1` argument to the binder is a placeholder telling `bind()` where arguments to the resulting function object should go. `g()`'s first argument is used as `f()`'s second

```
int pow(int,int);
double pow(double,double); // pow() is overloaded

auto pow2 = bind(pow,_1,2) // error: which pow()?
auto pow2 = bind((double(*)(double,double))pow,_1,2); // OK (ugly)
```

### `mem_fn()`

produces a function object that can be called as a nonmember function

```
void user(Shape* p)
{
  p−>draw();
  auto draw = mem_fn(&Shape::draw);
  draw(p);
}
```

Main use is when an algorithm requires an operation to be called as a non-member:

```
void draw_all(vector<Shape*>& v)
{
  for_each(v.begin(),v.end(),mem_fn(&Shape::draw));
}
```

Often, lambdas provide a simple alternative:

```
void draw_all(vector<Shape*>& v)
{
  for_each(v.begin(),v.end(),[](Shape* p) { p−>draw(); });
}
```

### `function`

`bind()` can be used directly, and it can be used to initialize an `auto` variable (it's like a lambda). A `function` is specified with a specific return type and a specific argument type:

```
int f1(double);
function<int(double)> fct {f1}; // initialize to f1
int f2(int);

void user()
{
  fct = [](double d) { return round(d); }; // assign lambda to fct
  fct = f1; // assign function to fct
  fct = f2; // error : incorrect argument type
}
```

## Type Functions

Evaluated at compile time given a type as argument or returning a type. For numerical types, `numerical_limits` from `<limits>` presents useful functions:

`constexpr float min = numeric_limits<float>::min(); // smallest positive float`

or

`constexpr int szi = sizeof(int); // the number of bytes in an int`

### `iterator_traits`

`sort()` takes a pair of iterators to define a sequence. They must be random access. Some containers (`forward_list`) do not offer that. `iterator_traits` allow us to check which kind of iterator supported.

(read)

### Type Predicates

```
bool b1 = Is_arithmetic<int>(); // yes, int is an arithmetic type

bool b2 = Is_arithmetic<string>(); // no, std::str ing is not an arithmetic typ
```

Found in `<type_traits>`. Other examples: `is_class`, `is_pod`, `is_literal_type`, `has_virtual_destructor`, `is_base_of`

> `static_assert(Is_arithmetic<Scalar>()` vs `return std::Is_arithmetic<T>::value`

older programs use `::value` directly unstead of `()`

```
Is_arithmetic<T>()        // OK
Is_arithmetic<T>::value  // Also OK
```

Without `()` or `::value` would not be trated as a boolean, just a type
