# Containers

Create collection of values and then manipulating these collections. Reading characters as a `string` then printing a `string` is an example. A class with the purpose of holding objects is a _container_.

## Vector

The most useful standard-library container. Contiguous elements stored in memory. A handle holding pointer to the first, one past the last element, and one past the last allocated space

Initialize a Vector:

```
vector<Entr y> phone_book = {
  {"David Hume",123456},
  {"Karl Popper",234567},
  {"Ber trand Ar thur William Russell",345678}
};
```

Accessing them:

```
void print_book(const vector<Entry>& book)
{
  for (int i = 0; i!=book.size(); ++i)
    cout << book[i] << '\n';
}
```

When defining a vector, give the initial size:

```
vector<int> v1 = {1, 2, 3, 4}; // size is 4

vector<string> v2; // size is 0

vector<Shape*> v3(23); // size is 23; initial element value: nullptr

vector<double> v4(32,9.9); // size is 32; initial element value: 9.9
```

Initial value is 0 for integers and `nullptr` for pointers. You can also specify the default value as second parameter!

> The initial size cannot be changed.

`push_back()` adds a new element to the end and increases its size by one:
`phone_book.push_back(e);`

The standard library `vector` has members `capacity()`, `reserve()` (make room for more elements), and `push_back()`. Implementing `push_back()`:

```
template<typename T>
void Vector<T>::push_back(const T& t)
{
  if (capacity()<size()+1) // make sure we have space for t
    reserve(siz e()==0?8:2 * size()); // double the capacity
  new(space){t}; // initialize *space to t
  ++space;
}
```

### Elements

Can just store a pointer (virtual functions):

```
vector<Shape> vs; // No, don’t --- there is no room for a Circle or a Smiley

vector<Shape*> vps; // better, but see 4.5.4

vector<unique_ptr<Shape>> vups; // OK

```

### Range Checking

Does not guarantee range checking.

> `book[book.size()]` out of range
> `book.at(_)` throws exception when out of range

> Also `resize()` allows changing size of vectors!

## List

Doubly-linked list: use this when want to insert and delete elements without moving others

```
list<Entry> phone_book = {
  {"David Hume",123456},
  {"Karl Popper",234567},
  {"Bertrand Arthur William Russell",345678}
};
```

Disadvantage: don't index. Have to iterate:

```
int get_number(const string& s)
{
  for (const auto& x : phone_book)
    if (x.name==s)
      return x.number;
  return 0; // use 0 to represent "number not found"
}
```

Every standard library container has functiond `begin()` and `end()` which return an iterator to the first and one-past-the-last element. So we can also write this function like:

```
int get_number(const string& s)
{
  for (auto p = phone_book.begin(); p!=phone_book.end(); ++p)
    if (p−>name==s)
      return p−>number;
  return 0; // use 0 to represent "number not found"
}
```

`insert(p,elem)` inserts an element with copy of value `elem` before the element pointed to by `p`. `erase(p)` removes the element pointed to by `p` and destroys it.

A `vector` performs traversal (`find()` and `count()`) and sorting and searching (`sort()` and `search()`)

## Map

Dictionary! Implemented as balanced binary tree.

> $\mathcal{O}(\log n)$ but are better for traversal and iterating keys in order. Different from $\mathcal{O}(1)$ hashmaps

```
map<string,int> phone_book {
  {"David Hume",123456},
  {"Karl Popper",234567},
  {"Bertrand Arthur William Russell",345678}
};
```

Can use `find()` and `insert()` instead of `[]`

## Unordered Map

The cost of `map` lookup is $\mathcal{O}(\log n)$. So often use hashing for faster operations (when order doesn't matter). This is what an unordered map is!

use `unordered_map` from `<unordered_map>`:

```
unordered_map<string,int> phone_book {
  {"David Hume",123456},
  {"Karl Popper",234567},
  {"Bertrand Arthur William Russell",345678}
};
```

## Container Overview

Look up the common container items. Here are some:

```
vector<T>
list<T>
forward_list<T> // singly-linked list
deque<T>
set<T>
map<K,V>
multimap<K,V> // a key can occur multiple times
unordered_map<K,V>
unordered_set<T>
```

Also `queue<T>`, `stack<T>`, `priority_queue<T>`. Also more specific fixed size containers: `array<T,N>`, `bitset<N>`

Basic operators in every container:

- `begin()` and `end()` give iterators to the first and one-beyond-the-last elements, respectively.
- `push_back()` can be used (efficiently) to add elements to the end of a `vector`, `list`, and other
  containers.
- `size()` returns the number of element

A `vector` is cheap and easy: usually more efficient than `list` for short sequences of small elements (even for `insert()` and `erase()`). Just use `vector` for some sequence of elements most of the time.

Consider `forward_list` optimized for the empty sequence (one word) when number of elements is zero or low (probably because have to save fewer pointers and fast to iterate small linked list, so only need singly linked).
