# Algorithms

Sort a vector:

```
void f(vector<Entry>& vec, list<Entry>& lst)
{
  sort(vec.begin(), vec.end());

  unique_copy(vec.begin(), vec.end(), lst.begin());
}
```

Must define `<` for `Entry`s to be able to sort:

```
bool operator<(const Entry& x, const Entry& y) // less than
{
  return x.name<y.name; // order Entrys by their names
}
```

Standard algo is expressed in terms of (half-open) sequence of elements. A _sequence_ is represented by a pair of iterators specifying the first element and beyond-the-last element with `end()`.

Example: `sort()` sorts the sequence defined from `vec.begin()` to `vec.end()`

## Use of Iterators

When encountering a container, a few iterators can be obtained (like `begin()` and `end()`). For example, `find` looks for a value in a sequence and returns an iterator to the element found:

```
bool has_c(const string& s, char c) // does s contain the character c?
{
  auto p = find(s.begin(),s.end(),c);

  if (p!=s.end())
    return true;
  else
    return false;
}
```

## Iterator Types

Many different types of iterators. For example, a `vector` iterator could be an ordinary pointer, or could be implemented as a pointer to the `vector` plus an index.

A `list` iterator must be more complicated because an element in a `list` doesn't know where it is: may point to a link.

> Applying `++` to an iterator yields an iterator that refers to the next element

> **`*` yields the element to which the iterator refers**

## Stream Iterators

`ostream_iterator` first needs to specify which stream will be used and the types of objects written to it:

`ostream_iterator<string> oo {cout}; // write strings to cout `

(read about it)

## Predicates

read about it

## Algorithm Overview

`<algorithm>` header in the `std` namespace.

> A half-open sequence from `b` to `e` is referred ot as `[b:e)`

```
p = find(b,e,x)
p = find_if(b,e,x)
sort(b,e)
sort(b,e,f) // specify criterion f
replace(b,e,v,v2)
p = copy(b,e,out)
```