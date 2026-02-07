# I/O Streams

`ostream` converts typed objects to a stream of characters (bytes)

## Output

In `<ostream>`, the I/O stream library defines output for every built-in type.

- `<<` is the "put to" operator: output objects of type `ostream`
  - `cout` is the standard output stream and `cerr` is the standard stream for reporting errors
- Values writtn to `cout` are converted to a sequence of characters

`cout << 10;` places the character 1 followed by 0 on the standard output stream.

`cout << "the value of i is " << 1 << '\n'`

## Input

In `<istream>`, the standard library offers `istream`s for input. Like `ostrem`s, `istream`s deal with character string representations of built-in types... can easily be extended

The operator `>>` "get from" is used as an input operator; `cin` is teh standard input stream:

```
int i;
cin >> i; // read integer into i

double d;
cin >> d;
```

Can also chain like outputs

```
int i;
double d;
cin >> i >> d;
```

```
void hello()
{
  cout << "Please enter your name\n";
  string str;
  cin >> str;
  cout << "Hello, " << str << "!\n";
}
```

Can also use `getline()` function:

```
void hello_line()
{
  cout << "Please enter your name\n";
  string str;
  getline(cin,str);
  cout << "Hello, " << str << "!\n";
}
```

## I/O State

An `iostream` has a state we can examine to determine if an operation succeeded. The most common is to read a sequence of values:

```
vector<int> read_ints(istream& is)
{
  vector<int> res;
  int i;
  while (is>>i)
    res.push_back(i);
  return res;
}
```

`is>>i` returns a reference to `is` and testing an `iostream` yields `true` if the stream is ready for another operation

## I/O of User-Defined Types

`iostream` allows programmers to define I/O for their own types. Consider type `Entry`:

```
struct Entry {
  string name;
  int number;
};
```

Use a `{"name", "number"}` format

```
ostream& operator<<(ostream& os, const Entry& e)
{
  return os << "{\"" << e.name << "\", " << e.number << "}";
}
```

A user-defined output operator takes its output stream (by ref) as first argument and returns it as result

## Formatting

We can explicitly set the output format for floating-point numbers:

```
constexpr double d = 123.456;

cout << d << "; " // use the default for mat for d

...<< scientific << d << "; " // use 1.123e2 style for mat for d

...<< hexfloat << d << "; " // use hexadecimal notation for d

...<< fixed << d << "; " // use 123.456 style for mat for f

...<< defaultfloat << d << '\n'; // use the default for mat for
```

## File Streams

`<fstream>` provides streams to and from a file:

- `ifstream`s for reading and writing from a file
- `ofstream`s for writing to a file
- `fstream`s for reading from and writing to a file

```
ofstream ofs("target"); // ‘‘o’’ for ‘‘output’’
if (!ofs)
  error("couldn't open 'target' for writing");
```

```
fstream ifs; // ‘‘i’’ for ‘‘input’’
if (!ifs)
  error("couldn't open 'source' for reading");
```

Assuming the test succeeded, `ofs` can be used as an ordinary `ostream` and `ifs` can be used as an ordinary `istream` (like `cin`)

## String Streams

In `<sstream>`, the standard library provides streams to and from a `string`:

- `istringstream`s for reading from a `string`
- `ostringstream`s for writing to a `string`
- `stringstream`s for reading from and writing to a `string`

Example:

```
void test()
{
ostringstream oss;
oss << "{temperature," << scientific << 123.4567890 << "}";
cout << oss.str() << '\n';
}
```
