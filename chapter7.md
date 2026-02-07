# Strings and Regular Expressions

## Strings

The `string` type provides many useful functions, like concatenation (use `+`)

A `string` is mutable (remember in C, character arrays are mutable but String literals are NOT)

- Python and Java strings are also immutable!

In C:

```
char s[] = "hello";
s[0] = 'H';   // OK
```

- `hello` is copied into a writable array

HOWEVER

```
char s[] = "hello";
s[0] = 'H';   // OK
```

- `hello` is in read-only memory (pointing to it, not a copy)

In C++:

```
std::string s = "hello";
s[0] = 'H';       // OK
s += " world";    // OK
```

- `std::string` manages its own heap buffer
- Safely mutable

However, C style strings still immutable

```
char s[] = "hello";     // mutable
const char* p = "hi";  // immutable
```

Python:

```
s = "hello"
s[0] = "H"      # ❌ TypeError
```

When you change a string, what really happens is...

```
s = s + "!"
```

a new string is created!

Java:

```
String s = "hello";
s.charAt(0) = 'H';   // ❌ impossible
```

String is immutable (same like Python)

...back to C++

The `substr()` operation returns a string _that's a copy_ of the substring indicated by arguments.

```
string name = "Niels Stroustrup";

string s = name.substr(6,10); // s = "Stroustrup"
```

The first argument tells where to start, the second is the length

`replace()` replaces a substring with a value (first arg is position, secon is length)

```
name.replace(0, 5, "nicholas") // name becomes "nicholas Stroustrup"
```

Index into strings with `[]` or `at()`

### `string` Implementation

Consider. strings

```
string s1 {"Annemarie"}; // short string

string s2 {"Annemarie Stroustrup"}; // long string
```

In memory, s1 is length 10 memory `Annemarie\0` and s2 is length 21 memory with `Annemarie Stroustrup\0`

When a string's value changes from short to long (and vice versa), its representation adjusts. For multi-thread apps, memory allocation can be costly. When many strings of diff lengths used, memory fragments can result. Short_string optimizations are common! To handle multiple character sets, `string` is an alias for `basic_string` template `char`:

```
template<typename Char>

class basic_string {
  // ... string of Char ...
};

using string = basic_string<char> // THIS IS THE ALIAS
```

`using Jstring = basic_string<Jchar>`

## Regular Expressions

`std::regex` powerful for string processing

`regex pat (R"(\w{2}\s∗\d{5}(−\d{4})?)"); // US postal code pattern: XXddddd-dddd and variants`

`(\w{2}\s∗\d{5}(−\d{4})?` specifies a pattern starting with two letters `\w{2}` optionally followed by some space `\s*` and five digits `\d{5}` and optionally a dash and four digits `-\d{4}`.

To express this pattern, use a _raw string literal_ with `R"( terminated by )"`. This allows backslashes and quotes to be used directly in the string

`regex pat {"\\w{2}\\s∗\\d{5}(−\\d{4})?"}; // U.S. postal code pattern`

`<regex>` standard library provides support for

- `regex_match()`: Match a regular expression against a string (of known size)
- `regex_search()`: Search for a string that matches a regular expression in an (arbitrarily long)
  stream of data
- `regex_replace()`: Search for strings that match a regular expression in an (arbitrarily long)
  stream of data and replace them
- `regex_iterator`: Iterate over matches and submatches
- `regex_token_iterator`: Iterate over non-matches

### Searching

The easiest way of using a pattern is to search for it in a stream:

```
int lineno = 0;

for (string line; getline(cin,line); ) { // read into line buffer
  ++lineno;
  smatch matches; // matched strings go here
  if (regex_search(line ,matches,pat)) // search for pat in line
  cout << lineno << ": " << matches[0] << '\n';
}
```

`regex_search(line, matches, pat)` searches for the `line` for anything that matches the regular expression in `pat` and stores them in `matches`. If no match found, return `false`.

### Regular Expression Notation

(Look at book)

### Iterators

`regex_iterator` iterates over a stream finding matches for a pattern. Example: output all whitespace-separated words in a `string`:

```
void test()
{
  string input = "aa as; asd ++eˆasdf asdfg";
  reg ex pat {R"(\s+(\w+))"};
  for (sreg ex_iterator p(input.begin(),input.end(),pat); p!=sregex_iterator{}; ++p)
  cout << (∗p)[1] << '\n';
}
```

`regex_iterator` is a bidirectional iterator, so cannot directly iterate over an `istream`. Can't write through `regex_iterator`
