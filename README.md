
# Introduction

Json-11 is yet another C++ class implementing [JSON](http://json.org) data interchange format. It is designed with the (currenly) latest C++11 standard in mind. The code is small (a single file less than 1000 lines in size), reasonably fast, and the interface (aka API) is made as simple as possible.

# Usage

    #include <json11.h>

Then drop json11.cpp into your project to compile. Note that you need a compiler that supports C++11. I only tested it with GCC 4.8.1 on Linux. For GCC don't forget to add `-std=c++11` to compiler flags.

## The basics

```c++
Json jnull; // creates a null Json object.
Json jtrue = true, pi = 3.14, str = "Hello";  // scalar objects
Json arr { 1, 2, str };   // array object, using initializer list
arr << false;  // do you prefer 'push_back' ?
```

To create key-value objects, start with a null object. Then use method `set(key,value)` to populate it.

```c++
    Json obj;
    obj.set("one", 1).set("pi", pi).set("arr", arr);
    cout << obj << endl;   // prints {"one":1,"pi":3.14,"arr":[1,2,"Hello",false]}
```

To extract scalar value from a Json instance, assign it to a scalar variable of matching type or use a cast. For collections use `operator[]` with either integer or string indexes.

```c++
    string text = str;     // text is Hello (without quotes)
    double num_pi = pi;
    int one = obj["one"];
    string hello = obj["arr"][2];
```

To 'stringify' a Json instance, write it to a std::ostream. To get it as a std::string use method `stringify()` or its alias `format()`:

    string jsontext = obj.format();

To parse a JSON text, read it from a std::istream. To parse it from std::string use static  method `parse()`:

```c++
    Json obj2 = Json::parse(jsontext);
    cout << obj == obj2 << endl;   // prints true; yes, Json_s_ are comparable
    ifstream fs("big.json");
    Json big(fs);
```

To determine the type and size of a Json instance use methods `type()` and `size()`. The first returns one of `enum Json::Type` values:

    enum Json::Type { JSNULL, BOOL, NUMBER, STRING, ARRAY, OBJECT };

The second returns either number of array elements or number of properties of an object. Property names are returned by `keys()` as `vector<string>`. These methods, together with `operator[]` allow to fully traverse an object.



## Exceptions

The Json class defines two exception classes, `parse_error` and `use_error`. The first one can be thrown only from `Json(istream&)`. The second is thrown when the class methods are used inappropriately, for example when `operator<<` is used for an object. It is also thrown when adding to a container would lead to circular dependency, like in this example:

```c++
    Json js1, js2;
    js1 << 1 << 2;
    js2 << js1;
    js1 << js2;   // error
```

The Json class methods can also throw standard exceptions out_of_range (from operator[]) and bad_cast.

## Constructors and initializers

<dl>
<dt>Json()</dt>
<dd>Default constructor creates a Json *null* instance.</dd>
<dt>Json(const Json&)</dt>
<dd>Copy contructor.</dd>
<dt>Json(Json&&)</dt>
<dd>Move contructor.</dd>
<dt>Json(std::istream&)</dt>
<dd>Reads and parses well-formed JSON UTF-8 text.</dd>
<dt>Json(T x)</dt>
<dd>Conversions from scalar type **T**, where **T** is one of bool, int, long, long long, float, double, long double, const char*, std::string. Internally all numbers are stored as long double.</dd>
<dt>Json& operator = (const Json&)</dt>
<dd>Copy initializer.</dd>
<dt>Json& operator = (Json&&)</dt>
<dd>Move initializer.</dd>
<dt>Json(std::initializer_list<Json>)</dt>
<dd>Creates an instance representing JSON array. Elements of the initializer_list must be either Json instances or convertible to Json.</dd>
<dt>virtual ~Json()</dt>
<dd>Destructor.</dd>
</dl>

## Casts

<dl>
<dt>Type type() const </dt>
<dd>Returns JSON type of this instance, one of enum Json::Type</dd>
<dt>operator T() const </dt>
<dd>If this instance represents a scalar value, returns it. T is one of bool, int, long, long long, float, double, long double, std::string. Throws std::bad_cast if wrong type is requested.</dd>
</dl>

## Arrays

<dl>
<dt>Json& operator << (const Json&)</dt>
<dd>Appends an element to the array. To create a new array just append something to a null instance, or use the constructor with initializer list.</dd>
<dt>const Json operator [] (int) const</dt>
<dd>Retrieves array element by index.</dd>
<dt>void insert(unsigned index, const Json& that)</dt>
<dd>Inserts *that* into array before *index*, so it becomes the one at *index*.</dd>
<dt>Json& replace(unsigned index, const Json& that)</dt>
<dd>Replaces array element at *index* by *that*</dd>
<dt>void erase(unsigned index)</dt>
<dd>Removes element at *index* from the array.</dd>
</dl>

These methods throw `use_error` if this Json instance is not an array. 

## Objects

<dl>
<dt>Json& set(std::string key, const Json& value)</dt>
<dd>Adds property "key:value" to this object, or replaces the *value* if *key* is already there. To create a new object, start from a null Json instance.</dd>
<dt>Json get(const std::string& key)</dt>
<dd>Returns value for the given key, or Json::null if this instance does not have such property.</dd>
<dt>const Json operator [] (std::string&)</dt>
<dd>Same as `get()`.</dd>
<dt>const Json operator [] (const char*)</dt>
<dd>Same as `get()`.</dd>
<dt>bool has(const std::string& key) const</dt>
<dd>Returns `true` if this instance has propery with given *key*.</dd>
<dt>std::vector<std::string> keys()</dt>
<dd>Returns all property keys for this instance.</dd>
</dl>

## Comparison

<dl>
<dt>bool operator == (const Json&)</dt> 
<dd>Compares Json instances. Scalars are equal if and only if they have same types and same values. Arrays are equal if their elements and order of them are the same. Objects are equal if their property keys and corresponding values are equal, regardless of order.</dd>
<dt>bool operator != (const Json&)</dt> 
<dd>The opposite.</dd>
</dl>

## Parsing and formatting

<dl>
<dt>static Json parse(const std::string&)</dt> 
<dd>Returns a Json instance built from well-formed JSON text. UTF-8 encoding is assumed. See also Json(std::istream&).</dd>
<dt>std::string stringify()</dt> 
<dd>Returns well-formed JSON text representing this object as a string.</dd>
<dt>std::string format()</dt> 
<dd>Same as stringify().</dd>
<dt>friend std::ostream& operator << (std::ostream&, const Json&)</dt> 
<dd>Writes well-formed JSON text representing this object into std::ostream.</dd>
<dt>static int indent</dt> 
<dd>If not 0, result of formatting looks prettier.</dd>
</dl>

## Etc.

<dl>
<dt>size_t size() const</dt> 
<dd>Returns size of an array or number of properties for an object.</dd>
<dt>static Json null</dt> 
<dd>The **null** instance.</dd>
</dl>

## Miscellaneous notes

Json class defines a static member Json::null of type JSNULL. It can be used to test for 'nullity' and for removing all contents from an object:

    if (js == Json::null)
        big_object = Json::null;

------

Reading a file creates a representation of entire file in memory. When it is not desirable, use static method `traverse`:

    Json::traverse(fs, on_entry, on_leave, on_value);

which calls supplied procedures while traversing through the file.

------

There is a difference between s1 and s2 in the following example:

    Json hello = "hello";
    string s1 = hello;
    string s2 = hello.format();

The assignment to s1 is a cast, so the value of s1 is _hello_. The value of s2 is _"hello"_ (with double quotes) because `format` creates a well-formed JSON text and returns it as a string.

------
