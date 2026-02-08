# Introduction to C++!

> Ethan's notes of _Tour of C++_

Most basic C++ function declaration (like Java)

> Elem\* next_elem();
>
> void exit(int);

## Understanding different signatures

When defining functions, an important component to signatures is passing arguments by reference vs passing by pointer. Also the idea of when to use `const`. You can also pass by value (copy).

> Example of passing by reference
>
> `void foo(BigObject& object)`

- No copy is made
- `object` must exist (cannot be null)
- Modifications affect the caller
- Preferred when the object is required

> Example of passing by pointer
>
> `void foo(BigObject* object)`

- No copy is made
- `object` can be null
- Explicitly signals optional ownership or presence
- Modifications affect the caller

Now, if we use const,

```
void foo(const BigObject& object);  // read-only, non-null

void foo(const BigObject* object);  // read-only, possibly null
```

> `const` is not just a hint, it actually prevents the object from being mutated.

> Passing by copy?
>
> `void foo(BigObject object);`

- A copy of object is made (copy ctor or move ctor)
- Changes inside foo do NOT affect the caller
- The parameter is always non-null
- Potentially expensive for large objects

So both passing by reference and pointer can mutate the original object

> This is an important topic which will be covered later, but `&` means alias (reference) and `*` is the pointer/address

## Basic C++ data types and operators

```
bool // built-in, unlike C
char
int
double // standrad double-precision floating point (64?)
unsigned // non-negative integer (can be higher)
```

> can use the `sizeof` opertor: `sizeof(char)=1` and `sizeof(int)=4` (size is in bytes)

```
x + y
+x // unary plus (identity)
-x // negation (unary minus)
(all standard)
...

//Logic operators
x&y // bitwise and
x|y // bitwise or
x^y // bitwise exclusive or (Xor)
x&&y // logic and
x||y // logic or
```

> don't need variable type when defining with `auto`! Ex: `auto b = true;`

```
// Pre and post-increment exist

int x = 5;
int y = ++x; // returns y = 6 but also x = 6 (pre increment)

int x = 5;
int y = x++; // returns the OLD value... y=5 and x=6 now! (post increment)
```

## Constants (again)

`const` indicates "I promise not to change this value"---the compiler will enforce this

`constexpr` means "to be evaluated at compile time."

`constexpr double square(double x) {return x*x};`

- Often use for actual constants: `const int dmv = 16;` or `constexpr double max1 = 1.4 * square(dmv);`

`constexpr` lets the compiler do the work early instead of at runtime and prove things about your program (SPEED).

## Pointers, arrays, references

Define an array of elements: `char v[6];`

Define a pointer to character: `char* p;`

`char* p = &v[3]; // p points to v's fourth element`

`char x = *p // *p is the object that p points to`

`*` means "contents of" and `&` means "address of"

Iterating over array and incrementing

```
void increment()
{
  int v[] = {0, 1, 2, 3};
  for (auto& x : v)
    ++x
}
```

Pass vector by reference: a reference is similar to a pointer (in terms of mutavility / no copy), but you don't need a prefix `*` to access the value

```
void sort(vector<double>& v)
```

If I still want to waste the cost of copying, but I don't want to modify the element, use `const` reference:

```
void sort(const vector<double>& v)
```

Null pointer

```
double* pd = nullptr;
Link<Record>* lst = nullptr;
int x = nullptr;
```

Passing argument by pointer (an array of characters)

```
int count_x(char* p, char x)
// count the number of occurrences of x in p[]
{
  if(p==nullptr) return 0;
  int count = 0;
  for(; p!=nullptr; ++p)
    if (*p==x)
      ++count;
  return count;
}

```

Can also do this with a while loop

```
int count_x(char* p, char x)
// count the number of occurrences of x in p[]
{
  int count = 0;
  while (p) {
    if (*p==x)
      ++count;
    ++p;
  }
  return count;
}
```

This still covers the scenario where `p` is empty and the poiner is null.

Can't do pass by reference for the above: `char& p` will be the actual first character (can't increment through this like an address).

> `while(p)` is like `while(p!=nullptr)`

## Tests

```
bool accept()
{
  cout << "Do you want to proceed (y or n)?\n";

  char answer = 0;

  cin >> answer; // read answer

  if (answer == 'y')
    return true;

  return false;
}
```

`<<` means "put to" and `>>` means "get from". `cin` is the standard input stream.

Switch statements work in C++!

```
void action()
{
  while (true) {
    cout << "enter action:\n";

    string act;
    cin >> act;

    Point delta {0,0};

    for (char ch : act) {
      switch (ch) {
      case 'u': // up

      case 'n': // north
        ++delta.y;
        break;

      case 'r': // right
      case 'e': // east
        ++delta.x;
        break;

      // ... more actions ..
    }
}
```

## Additional?

Pointer syntax: `p->x` is same as `(*p).x` (for classes)

Pointer checks?
`if (!p) {...} // pointer null check`
`if (!x) {...} // boolean false`

**Some trivia:**

> `Pointers vs. References:` A pointer stores a memory address and can be null or reassigned, while a reference is an alias for an existing object and cannot be null or reassigned after initialization.

> `Stack vs. Heap Memory:` Memory on the stack is allocated and deallocated automatically as variables go in and out of scope. Heap memory is dynamically managed using new and delete (or smart pointers) and persists until explicitly deallocated, offering more control but requiring careful management.

> `const` and `static` keywords: const makes a variable or object immutable after initialization. `static` changes the storage duration for local variables (persisting across function calls) or limits the visibility of global variables and functions to the current file.

> `this` pointer: A hidden pointer in non-static member functions that points to the current object instance, used to access members or return the object itself.
