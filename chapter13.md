# Concurrency

Improve throughput and responsiveness. Atomic operations allow lock-free programming. Memory model ensures as long as a programmer avoids data races, everything works as expected.

Overview of `thread`s, `mutex`es, `lock()` operations, `packaged_task`s, and `future`s.

## Tasks and `thread`s

_task_ := computation that can potentially be executed concurrently with others

_thread_ := system-level representation of a task in a program

A task to be executed concurrently with others is launched by constructing a `std::thread` found in `<thread>` with the task as its argument:

```
void f();

struct F {
  void operator()(); // F's call to operator
}

void user()
{
  thread t1 {f}; // f() executes in separate thread
  thread t2 {F()}; // F()() executes in separate

  t1.join(); // wait for t1
  t2.join(); // wait for t2
}
```

`join()`s ensure we don't exit `user()` until the threads have completed. To "join" a `thread` means to wait for it to terminate.

Concurent programming can be hard:

```
void f() { cout << "Hello"; }

struct F {
  void operator()() { cout << "Parallel World!\n"; }
};
```

> This is a bad error. Here, `f` and `F()` each use `cout` without synchronization. This could produce a weird output like "PaHeralllel o World!"

Our goal is to keep tasks scompletely separate except for where they communicate. We have to pass args, get results, andmake sure there is no use of shared data in between (data races).

## Passing Arguments

A task needs data to work upon. We can easily pass data (or pointers or references to data) as arguments.

```
void f(vector<double>& v); // function do something with v

struct F { // function object: do something with v
  vector<double>& v;
  F(vector<double>& vv) :v{vv} { }
  void operator()(); // application operator
};

int main()
{
  vector<double> some_vec {1,2,3,4,5,6,7,8,9};
  vector<double> vec2 {10,11,12,13,14};

  thread t1 {f,ref(some_vec)}; // f(some_vec) executes in a separate thread
  thread t2 {F{vec2}}; // F(vec2)() executes in a separate thread

  t1.join();
  t2.join();
}
```

> What is this `()()`? This means construct an argument, then immediately call it like an argument. `operator()` immediately turns `F` into a function object (a functor). So now this can be called: sugar for `f.operator()()`

`F{vec2}` saves a reference to the argument vector `F`. `F` can now use that vector and hopefully no other task accesses `vec2` while `F` is executing. Passing `vec2` by value would eliminate that risk.

The initialization with `{f,ref(some_vec)}` uses a `thread` variadic template constructor that can accept an arbitrary sequence of arguments. The `ref()` is a type function from `<functional>` that is needed to tell the variadic template to treat `some_vec` as a reference, rather than an object.

## Returning Results

Can pass input data by `const` reference and to pass the location of a place to deposit the result as separate argument. This is vs passing non-`const`.

```
void f(const vector<double>& v, double∗ res); // take input from v; place result in *res

class F {
public:
  F(const vector<double>& vv, double∗ p) :v{vv}, res{p} { }
  void operator()(); // place result in *res

private:
  const vector<double>& v; // source of input
  double* res; // target for output
};

int main()
  {
  vector<double> some_vec;
  vector<double> vec2;

  // ...

  double res1;
  double res2;

  thread t1 {f,cref(some_vec),&res1}; // f(some_vec,&res1) executes in a separate thread
  thread t2 {F{vec2,&res2}}; // F{vec2,&res2}() executes in a separate thread

  t1.join();
  t2.join();

  cout << res1 << ' ' << res2 << '\n';
}
```

Not so common but works.

## Sharing Data

Access has to be synchronized so at most one task at a time has access. The solution is a `mutex`. A `thread` acquires a mutex using a `lock()` operation:

```
mutex m;
int sh; // shared

void f()
{
  unique_lock<mutex> lck {m}; // acquire mutex
  sh += 7; // manipulate shared data
  // release mutex implcitly
}
```

`unique_lock` constructor acquires the mutex (using `m.lock()`). If another thread tries to access, it blocks and waits until `m.unlock()`. When `mutex` is released, `thread`s waiting resume.

Might need several resources simultaneously. This is deadlock.

> Example: `t1` acquires `m1` and tries to acquire `m2` while `t2` acquires `m2` and then tries to acquire `m1`---neither task proceeds...

Working with several locks:

```
void f()
{
  // ...
  unique_lock<mutex> lck1 {m1,defer_lock}; // defer_lock: don’t yet try to acquire the mutex
  unique_lock<mutex> lck2 {m2,defer_lock};
  unique_lock<mutex> lck3 {m3,defer_lock};
  // ...
  lock(lck1,lck2,lck3); // acquire all three locks
  // ... manipulate shared data ...
} // implicitly release all mutexes
```

`lock()` will proceed only after acquiring all its `mutex` argument and will never block/go to sleep while holding `mutex`. Destrctors for `unique_lock`s ensure `mutex`es are released when a `thread` leaves the scope.

## Waiting for Events

Sometimes, a `thread` has to wait for some external event, such as another `thread` completing; the simplest event is time passing. Using `<chrono>`:

```
using namespace std::chrono;

auto t0 = high_resolution_clock::now();
this_thread::sleep_for(milliseconds{20});
auto t1 = high_resolution_clock::now();

cout << duration_cast<nanoseconds>(t1−t0).count() << " nanoseconds passed\n";
```

Don't even have to launch a `thread`; by default, `this_thread` refers to the one and only thread.

> `duration_cast` adjusts clock's units to nanoseconds

> A `condition_variable` allows one `thread` to wait for another: this supports efficient sharing

## Communicating Takss

Operate at conceptual level of tasks rather than lower level of threads and locks

- `future` and `promise` for returning a value from a task spawned on a separate thread
- `package_task` to help launch tasks and connect up the mechanisms for returning a result
- `async()` for launching of a task in a manner similar to calling a function

These facilities are found in `<future>`

### `future` and `promise`

Enable a transfer of a value between two tasks without explicit use of a lock: the system implements the transfer. Idea: when a task wants to pass a value to another, it puts the value into a `promise`; makes the value appear in the corresponding `future`

If we have `future<X>` called `fx` we can `get()` the value of type `X` from it: `X v = fx.get();`

### `packaged_task`

How to get `future` into a task that needs that result and the corresponding `promise` into a thread that shoulf produce that result? `packaged_task` simplifies `future`s and `promise`s to be run on `thread`s. Wrapper code to put the return value or exception from the task into a `promise`

```
double accum(double∗ beg, double∗ end, double init)
  // compute the sum of [beg:end) starting with the initial value init
{
  return accumulate(beg,end,init);
}

double comp2(vector<double>& v)
{
  using Task_type = double(double∗,double∗,double); // type of task

  packaged_task<Task_type> pt0 {accum}; // package the task (i.e., accum)
  packaged_task<Task_type> pt1 {accum};

  future<double> f0 {pt0.get_future()}; // get hold of pt0’s future
  future<double> f1 {pt1.get_future()}; // get hold of pt1’s future

  double* first = &v[0];
  thread t1 {move(pt0),first,first+v.size()/2,0}; // start a thread for pt0
  thread t2 {move(pt1),first+v.size()/2,first+v.size(),0}; // start a thread for pt1

  // ...

  return f0.get()+f1.get(); // get the results
}
```

### `async()`

Used to launch tasks to potentially run asynchronously:

```
double comp4(vector<double>& v)
  // spawn many tasks if v is large enough
{
  if (v.siz e()<10000) // is it worth using concurrency?
    return accum(v.begin(),v.end(),0.0);

  auto v0 = &v[0];
  auto sz = v.siz e();
  auto f0 = async(accum,v0,v0+sz/4,0.0); // first quarter
  auto f1 = async(accum,v0+sz/4,v0+sz/2,0.0); // second quarter
  auto f2 = async(accum,v0+sz/2,v0+sz*3/4,0.0); // third quarter
  auto f3 = async(accum,v0+sz*3/4,v0+sz,0.0); // four th quar ter

  return f0.get()+f1.g et()+f2.g et()+f3.g et(); // collect and combine the results
}
```

Separates the "call part" of a function call from the "get result part," and separates both from the actual task. Using `async()` you don't have to think about threads and locks.

https://www.youtube.com/watch?v=4twJD5ezkag

Parallel computing is common with OpenMP!
