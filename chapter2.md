# User-Defined Types

## Structures

Define some vector type

```
struct Vector {
  int sz;
  double* elem; \\ pointer to elements
}
```

`Vector v;`

But this is just an empty vector. Construct the vector:

```
void vector_init(Vector& v, int s)
{
  v.elem = new double[s]; // allocate array of s doubles

  v.sz = s;
}
```

Call this with `vector_init(v,s);`

Notice how I pass in reference to directly mutate object (and no pointer because pointers are anoying).

`new` allocates memory from free storage or dynamic memory (heap)... destroyed with `delete` operator.

Use `.` to access struct members through name and `->` to access struct members through pointer:

```
void f(Vector v, Vector& rv, Vector* pv)
{
  v.sz;
  rv.sz;
  pv->sz;
}
```

This is how we delete a new vector created from heap:

```
void vector_free(Vector& v)
{
    delete[] v.elem;   // free heap memory
    v.elem = nullptr;  // avoid dangling pointer
    v.sz = 0;
}
```

Notice `delete[]` because we created with `new T[]`

## Classes

Public and private members are separated well

```
  class Vector {
  public:
    Vector(int s) :elem{new double[s]}, sz{s} { } // constr uct a Vector
    double& operator[](int i) { return elem[i]; } // element access: subscripting
    int size() { return sz; }

  private:
    double∗ elem; // pointer to the elements
    int sz; // the number of elements
  };
```

Define a new vector: `Vector v(6);`

What this actually does?

- v is an object with automatic storage (stack-like lifetime)
- Constructor runs -> allocates elem on the heap
- Destructor runs automatically when v goes out of scope
- Memory is cleaned up safely

Compare this to `Vector* v = new Vector(6);`

- The Vector object itself lives on the heap
- Constructor runs
- You must later call: `delete v;`

## Unions

A struct where all members allocated at the same address. It occupies only as much space as its largest member.

Consider a symbol table entry that holds a name and value:

```
enum Type { str, num };
struct Entry {
  char∗ name;
  Type t;
  char∗ s; // use s if t==str
  int i; // use i if t==num
};

void f(Entry∗ p)
{
  if (p−>t == str)
    cout << p−>s;
  // ...
}
```

`s` and `i` can never be used at the same time, so space is wasted. Specify that both should be members of a union:

```
union Value {
  char* s;
  int i;
}
```

And specity the value is held by a union:

```
struct Entry {
  char∗ name;
  Type t;
  Value v; // use v.s if t==str; use v.i if t==num
};

void f(Entry∗ p)
{
  if (p−>t == str)
    cout << p−>v.s;
  // ...
}
```

## Enumerations

`enum class Color {red, blue, green};`
`enum class Traffic_light {green, yellow, red};`

`Color col = Color::red`
`Traffic_light light = Traffic_light::red`

Type checking:

```
Color x = red; // error : which red?

Color y = Traffic_light::red; // error : that red is not a Color

Color z = Color::red; // OK!

int i = Color::red; // error : Color ::red is not an int

Color c = 2; // error : 2 is not a Color
```

`enum` class has only assignment, initialization, and comparisons:

```
Traffic_light& operator++(Traffic_light& t)
  // prefix increment: ++
{
  switch (t) {
  case Traffic_light::green:
    return t=Traffic_light::yellow;

  case Traffic_light::yellow:
    return t=Traffic_light::red;

  case Traffic_light::red:
    return t=Traffic_light::green;
  }
}
Traffic_light next = ++light; // next becomes Traffic_light::green
```

Note the above returns a reference! Not a copy or pointer.

> Can get a plain enum removing `class` from `enum class`

```
enum Color {red, green, blue};
int col = green; // col gets the value 1
```

> This implicitly converts them to their INTEGER values
