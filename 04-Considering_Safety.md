# Considering Safety


## Const as Much as Possible
`const` tells the compiler that a variable or method is immutable. This helps the compiler optimize the code and helps the developer know if a function has a side effect. Also, using `const &` prevents the compiler from copying data unnecessarily. The  [comments on `const` from John Carmack](http://kotaku.com/454293019) are also a good read.

```cpp
// Bad Idea
class MyClass
{
public:
  void do_something(int i);
  void do_something(std::string str);
};


// Good Idea
class MyClass
{
public:
  void do_something(const int i);
  void do_something(const std::string &str);
};

```

### Carefully Consider Your Return Types

 * Getters
   * Returning by `&` or `const &` can have significant performance savings when the normal use of the returned value is for observation
   * Returning by value is better for thread safety and if the normal use of the returned value is to make a copy anyhow, there's no performance lost
 * Temporaries and local values
   * Always return by value.


references: https://github.com/lefticus/cppbestpractices/issues/21 https://twitter.com/lefticus/status/635943577328095232 

### Do not pass and return simple types by const ref 

```cpp
// Very Bad Idea
class MyClass
{
public:
  explicit MyClass(const int& t_int_value)
    : m_int_value(t_int_value)
  {
  }
  
  const int& get_int_value() const
  {
    return m_int_value;
  }

private:
  int m_int_value;
}
```

Instead, pass and return simple types by value. If you plan not to change passed value, declare them as `const`, but not `const` refs:

```cpp
// Good Idea
class MyClass
{
public:
  explicit MyClass(const int t_int_value)
    : m_int_value(t_int_value)
  {
  }
  
  int get_int_value() const
  {
    return m_int_value;
  }

private:
  int m_int_value;
}
```

Why? Because passing and returning by reference leads to pointer operations instead by much more faster passing values in processor registers.

## Avoid Raw Memory Access

Raw memory access, allocation and deallocation, are difficult to get correct in C++ without [risking memory errors and leaks](http://blog2.emptycrate.com/content/nobody-understands-c-part-6-are-you-still-using-pointers). C++11 provides tools to avoid these problems.

```cpp
// Bad Idea
MyClass *myobj = new MyClass;

// ...
delete myobj;


// Good Idea
auto myobj = std::make_unique<MyClass>(constructor_param1, constructor_param2); // C++14
auto myobj = std::unique_ptr<MyClass>(new MyClass(constructor_param1, constructor_param2)); // C++11
auto mybuffer = std::make_unique<char[]>(length); // C++14
auto mybuffer = std::unique_ptr<char[]>(new char[length]); // C++11

// or for reference counted objects
auto myobj = std::make_shared<MyClass>(); 

// ...
// myobj is automatically freed for you whenever it is no longer used.
```

## Use `std::array` or `std::vector` Instead of C-style Arrays

Both of these guarantee contiguous memory layout of objects and can (and should) completely replace your usage of C-style arrays for many of the reasons listed for not using bare pointers.

Also, [avoid](http://stackoverflow.com/questions/3266443/can-you-use-a-shared-ptr-for-raii-of-c-style-arrays) using `std::shared_ptr` to hold an array. 

### Special Considerations for Real-Time Code

It is not advisable to allocate and deallocate memory in a Real-Time audio processing context. Make sure to preallocate space for vectors before with 

```
// Bad Idea
void process(mha_wave_t* s){
std::vector<float> myVect;
for(int i=0;i<128;++ii)
myVect.push_back(ii);
}

// Better Idea
class my_plugin_class {
std::vector<float> myVect;

void my_plugin_class(){ //ctor
myVect.reserve(128); // I need to store 128 floats.
}
};
void process(mha_wave_t* s){
for(int i=0;i<128;++ii)
myVect[ii]=ii;
}
```
## Use Exceptions

Exceptions cannot be ignored. Return values, such as using `boost::optional`, can be ignored and if not checked can cause crashes or memory errors. An exception, on the other hand, can be caught and handled. Potentially all the way up the highest level of the application with a log and automatic restart of the application.

Stroustrup, the original designer of C++, [makes this point](http://www.stroustrup.com/bs_faq2.html#exceptions-why) much better than I ever could.

### Special Considerations for Real-Time Code

Exceptions are not real-time safe. Use them very judiciously in real-time code.

## Use C++-style cast instead of C-style cast
Use the C++-style cast (static\_cast<>, dynamic\_cast<> ...) instead of the C-style cast. The C++-style cast allows more compiler checks and is considerably safer.

```cpp
// Bad Idea
double x = getX();
int i = (int) x;

// Not a Bad Idea
int i = static_cast<int>(x);
```
Additionally the C++ cast style is more visible and has the possibility to search for.

But consider refactoring of program logic (for example, additional checking on overflow and underflow) if you need to cast `double` to `int`. Measure three times and cut 0.9999999999981 times.
