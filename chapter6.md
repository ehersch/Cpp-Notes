# Library Overview

**Some big libraries**: string, ostream, vector, map, unique_ptr, thread, regex, complex

## Standard Library Components

- Run-type language support (allocation and run-time type information)
- The C standard library
- Strings
- I/O streams
- Framework of containers (`vector` and `map`) and algorithms (`find()`, `sort()`, `merge()`)
- Support for concurrency with `thread`s
- Special purpose containers like `array`, `bitset`, `tuple`

## Standard-Library Headers and Namespaces

```
#include<string>
#include<list>
```

To use standard library facilities, use `std::` prefix:

`std::string s {"Hello!"};`

To make it simpler, don't use `std::` and `#include` appropriate headers:

```
#include<string>
using namespace std;

string s{"Hello!"};
```

(Check textbook for standard library headers)

Headers from the C library (`<stdlib.h`) are provided.
