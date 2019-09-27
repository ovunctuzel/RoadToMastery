
<p align="center">
  <a href="">Programming</a> |
  <a href="unreal.html">Unreal</a> |
  <a href="misc.html">Misc</a> |
  <a href="soon.html">Coming Soon</a>
</p>

* * *

## C++ Study Journal

This page contains a list of quick tips and tutorials for programming and game development. This page mostly focuses on C++, but resources on C#, Python, Unreal Engine, Unity, and Math can also be found.
 
## Article List

### Programming

* [Endl is evil](#endl-is-evil-but-only-a-little-bit)
* [Accessing parent class members](#accessing-parent-class-members)
* [Data Alignment](#data-alignment)
* [Method Chaining](#method-chaining)
* [Using](#using)
* [Serialization](#serialization)
* [Trailing Return Types](#trailing-return-types)
* [Regex](#regex)
* [Raw Strings](#raw-strings)
* [Foreach Mutability](#foreach-mutability)
* [Forward Declarations](#forward-declarations)
* [Interesting Data Structures](#interesting-data-structures)
* [Memory & Smart Pointers](#memory-and-smart-pointers)
* [Explicit Template Instantiation](#explicit-template-instantiation)

### Coming Soon
* Memset and friends
* Garbage collection
* Better exception handling
* Implementation of C++ data structures
* Rvalue Lvalue
* Better multithreading, std::atomic

## Programming Tips

### Endl is evil, but only a little bit
Std::endl is equivalent to ‘\n’ followed by a std::flush. Flushing is emptying the stream buffer, and it is not always desired. Not using std::endl everywhere can slightly improve performance.

### Accessing Parent Class Members
If class B inherits from class A, B can call a function defined on A, or use a member inherited from A in the following way:
```
B::printStuffFromA()
{ 	
	A::print();
	print(); // print() has different implementations in A and B
	std::cout << x << A::x; // X is defined in both A and B
}
```

Use the base keyword in C#:
```
printStuffFromA()
{
	base.print();
	print();
	Console.WriteLine(base.x);
	Console.WriteLine(x);
}
```

### Data Alignment
In a C++ struct, data is padded with zeros to fit things in words. A struct with a char, an int, and a short takes up more space than a struct with an int, a short and a char. A good practice is to order members from large to small. However, unless working with severe hardware constraints or trying to optimize networked software, this only becomes relevant for millions of structs. More info: https://jonasdevlieghere.com/order-your-members/

### Method Chaining
Also known as Fluent APIs, method chaining can sometimes be used to create user friendly interfaces:
```
   GlutApp app(argc, argv);
   app.setDisplayMode(GLUT_DOUBLE|GLUT_RGBA|GLUT_ALPHA|GLUT_DEPTH);
   app.setWindowSize(500, 500);
   app.setWindowPosition(200, 200);
   app.setTitle("My OpenGL/GLUT App");
   app.create();
```
Above is an example from Glut, an OpenGL library. Several lines were used to setup a window using five different functions.

```
   FluentGlutApp(argc, argv)
       .withDoubleBuffer().withRGBA().withAlpha().withDepth()
       .at(200, 200).across(500, 500)
       .named("My OpenGL/GLUT App")
       .create();
```
Above is a fluent way of writing similar code. Note that each one of these functions must return the object itself so that they can be chained like this. In C++, ```return this``` would return a pointer to the class this statement exists in, so ```return *this``` can be used to return the object itself.

We can construct a fluent wrapper around a class to achieve this functionality cleanly. In the above example FluentGlutApp is a wrapper around  GlutApp(actually inherits GlutApp), and its functions call the necessary functions on GlutApp and return itself.

### Using
Using is commonly used for importing namespaces:
```
using Wow::This::Namespace::Is::Long
```
Using can also be used as an alias to shorten long types:
```
using Settings = std::map<std::string, std::vector<int>>;
Settings settings;
```

### Interesting Data Structures
#### C#

[Hybrid Dictionary](https://docs.microsoft.com/en-us/dotnet/api/system.collections.specialized.hybriddictionary?redirectedfrom=MSDN&view=netframework-4.8): Uses a linked list when the container size is small, automatically switches to a hash table implementation when the container gets sufficiently large.

#### C++
Coming soon...

### Serialization
We can serialize data (usually an object) to save them and possibly access them at a later time. We can either serialize in human readable formats (e.g. JSON), or more efficient machine formats (e.g. binary). Below is a practical examples:

#### C# Binary Serialization:
```
    public static class Serializer
    {
        public static void Save(string filePath, object objToSerialize)
        {
            try
            {
                using (Stream stream = File.Open(filePath, FileMode.Create))
                {
                    BinaryFormatter bin = new BinaryFormatter();
                    bin.Serialize(stream, objToSerialize);
                }
            }
            catch (IOException)
            {
            }
        }

        public static T Load<T>(string filePath) where T : new()
        {
            T rez = new T();

            try
            {
                using (Stream stream = File.Open(filePath, FileMode.Open))
                {
                    BinaryFormatter bin = new BinaryFormatter();
                    rez = (T)bin.Deserialize(stream);
                }
            }
            catch (IOException ex)
            {
                throw ex;
            }

            return rez;
        }
    }
```
Note that we need to import System.Runtime.Serialization.Formatters.Binary, and put [Serializable] tag on objects. 

#### Links:

[C# JSON Serialization](https://docs.microsoft.com/en-us/dotnet/api/system.runtime.serialization.json?view=netframework-4.8)

[C# Binary Serialization](https://docs.microsoft.com/en-us/dotnet/api/system.runtime.serialization.formatters.binary.binaryformatter?view=netframework-4.8)

[C++ Lightweight Serialization Library](http://uscilab.github.io/cereal/)

[Python Pickle](https://docs.python.org/3/library/pickle.html)

### Trailing Return Types

```
template<class T>
decltype(a*b) mul(T a, T b)
{
	return a*b;
}

template<class T>
auto mul(T a, T b) -> decltype(a*b)
{
	return a*b;
}
```

In the first example, the compiler doesn't know what type ```a*b``` is. In the second example, the compiler can deduce it thanks to trailing return types. More info: [IBM](https://www.ibm.com/developerworks/community/blogs/5894415f-be62-4bc0-81c5-3956e82276f3/entry/introduction_to_the_c_11_feature_trailing_return_types?lang=en)

### Regex
#### Regex in C++
```regex_match()``` returns true if the regular expression is a match against the given string otherwise it returns false.

```regex_search()``` searches for a pattern matching the regular expression in a string. Looks for substrings that match the pattern.

```regex_replace()``` replaces the pattern matching to the regular expression with a string.

Below is a simple C++ example:
```
#include <iostream>
#include <regex>
using namespace std;

int main()
{
	string str = "This is a generic string!";
	regex reg("This is a [a-g]+");
	bool searchMatches = regex_search(str, reg); // Match
	bool matchMatches = regex_match(str, reg); // No Match

	// Search for a substring
	smatch mat;
	regex_search(str, mat, reg);
	for(auto c : mat)
		cout << c; // Prints 'This is a ge'

	// Replace substring
	string result;
	regex_replace(back_inserter(result), str.begin(), str.end(), reg, "XYZ");
	cout << "\n" << result // Prints 'XYZneric string!'
}

```

More to come on regex patterns...

[Excellent Resource](https://regexr.com/)

### Raw Strings
Below strings ignore all escape characters. This is especially useful when constructing regex patterns.
#### C#
```string pattern = @"stage\.effect\.";```
#### C++
```std::string s = R"(This\is\a\raw\string\literal)"; ```

### Foreach Mutability
#### C#
In C#, we cannot directly assign to the value we iterate in a foreach loop, however, we can change the contents of the data structure. Let listlist be a list of integer lists:

```
    var listlist = new List<List<int>> { new List<int>{ 1, 2, 3, 4, 5 }, new List<int>{ 1, 2, 3, 4 } };

    // Legal - Actually modifies the inner lists
    foreach (var list in listlist)
    {
	list[1] = 99;
    }

    // Illegal - Compile Error
    foreach (var list in listlist)
    {
	list = new List<int> { 5, 5 };
    }

    // Illegal - Compile Error, cannot directly assign to list
    foreach (var list in listlist)
    {
	list = list.Reverse;
    }

    // Legal - Reverse using an inner for loop
    foreach (var list in listlist)
    {
	for (int i = 0; i < list.Count/2; i++)
	{
	    var temp = list[i];
	    list[i] = list[list.Count - i - 1];
	    list[list.Count - i - 1] = temp;
	}
    }
```

#### C++
In C++, range based for loops are mutable unless passed using const.
```
	std::vector<int> vec = { 1, 2, 3, 4, 5 };

	// Passes by value, vec is not modified
	for (int i : vec)
	{
		i++;
	}	

	// Passes by ref, vec is modified
	for (int & i : vec)
	{
		i++;
	}
```

### Forward Declarations
```
class ClassA;

class ClassB
{
public:
	ClassA* Class1Ptr;
};

```
In the above example we forward declare ClassA, so the compiler doesn't freak out when it sees the ClassA* variable Class1Ptr. We could achieve the same effect by doing an include "ClassA.h", but then ClassA has to be compiled, and can significantly increase build time. Forward declarations will only work if we have pointers to a class, so the below code won't compile:

```
class ClassA;

class ClassB
{
public:
	ClassA Class1Obj;
};

```
An additional benefit of forward declaration is being able to have classes that reference each other (no cyclic includes). No apparent downsides.

### Explicit template instantiation
Templated member functions must be defined in the header file, otherwise linker errors will occur. The reason is that templated functions are not compiled. If it is really necessary to define a templated function in a cpp file we need to explicitly instantiate them in the cpp file.

Header:
```
class BooleanExpressionParser
{
public:
	BooleanExpressionParser();

	template <typename T>
	void test(T t);
};

```
Source:
```
template <typename T>
void BooleanExpressionParser::test(T t)
{
	// Do stuff
	return;
}

// End of source file
template void BooleanExpressionParser::test(int);
```
Here we explicitly tell the compiler to create a version of this templated function using int as the type T.

### Memory and Smart Pointers
Most of this section comes from [Fluent C++](https://www.fluentcpp.com/2017/08/22/smart-developers-use-smart-pointers-smart-pointers-basics/).
#### Stack vs Heap
Everytime we declare a variable in C++, or pass a variable by value to a function, make a function call, etc. things get pushed into the stack. This means they are stored in consecutive(?) memory. **Objects allocated on the stack are automatically destroyed when they go out of scope.**

Dynamically allocated objects are stored on the heap.
```
int * pi = new int(42);
```
The new keyword returns a pointer, in this case it is an int pointer. Objects allocated on the heap **are not destroyed automatically**.

```
House* buildAHouse();
```
This function returns a House pointer. As the user of this function perhaps I should delete the pointer after I get my house. But what if I delete it and someone else also tries to delete it (undefined behavior)? If I don't delete it, and no one else does, perhaps there will be a memory leak.

RAII comes to the rescue. One simple rule: Wrap a resource (a pointer) into an object, and dispose of the resource in its destructor. This way the wrapper object is allocated in stack, and is automatically destroyed (delete ptr is called with the destructor) when it goes out of scope.

### Statically Linking Runtime Libraries
Quick recap on static vs. dynamic libraries (dll vs lib) from Stack Overflow by Charles E. Grant:

> There are static libraries (LIB) and dynamic libraries (DLL).
>
> Libraries are used because you may have code that you want to use in many programs. For example if you write a function that counts the number of characters in a string, that function will be useful in lots of programs. Once you get that function working correctly you don't want to have to recompile the code every time you use it, so you put the executable code for that function in a library, and the linker can extract and insert the compiled code into your program. Static libraries are sometimes called 'archives' for this reason.
>
> Dynamic libraries take this one step further. It seems wasteful to have multiple copies of the library functions taking up space in each of the programs. Why can't they all share one copy of the function? This is what dynamic libraries are for. Rather than building the library code into your program when it is compiled, it can be run by mapping it into your program as it is loaded into memory. Multiple programs running at the same time that use the same functions can all share one copy, saving memory. In fact, you can load dynamic libraries only as needed, depending on the path through your code. No point in having the printer routines taking up memory if you aren't doing any printing. On the other hand, this means you have to have a copy of the dynamic library installed on every machine your program runs on. This creates its own set of problems.
>
> As an example, almost every program written in 'C' will need functions from a library called the 'C runtime library, though few programs will need all of the functions. The C runtime comes in both static and dynamic versions, so you can determine which version your program uses depending on particular needs.

In Visual Studio, the C++ runtime library can be changed from dynamic to static from the project properties > C/C++ > Code Generation > Runtime Library > Multi-Threaded instead of Multi-Threaded DLL. This way the executable won't need any runtime dlls, but the exe would be larger. Make sure to do the same thing to other libraries/projects the exe project includes.



