## C++ Study Journal

This page contains a list of quick tips and tutorials for programming and game development. This page mostly focuses on C++, but resources on C#, Python, Unreal Engine, Unity, and Math can also be found.
 
## Article List

### Programming

* [Endl is evil](#endl-is-evil-but-only-a-little-bit)
* [Data Alignment](#data-alignment)
* [Method Chaining](#method-chaining)
* [Using](#using)
* [Serialization](#serialization)
* [Trailing Return Types](#trailing-return-types)
* [Regex](#regex)
* [Raw Strings](#raw-strings)
* [Foreach Mutability](#foreach-mutability)
* [Interesting Data Structures](#interesting-data-structures)

### Unreal
* [Replication Tips](#replication-tips)

### Coming Soon
* Memset and friends
* Garbage collection
* Better exception handling
* Implementation of C++ data structures
* Rvalue Lvalue
* Better Smart Pointers
* Better multithreading, std::atomic

## Programming Tips

### Endl is evil, but only a little bit
Std::endl is equivalent to ‘\n’ followed by a std::flush. Flushing is emptying the stream buffer, and it is not always desired. Not using std::endl everywhere can slightly improve performance.

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

## Unreal
### Replication Tips
We can replicate variables using ReplicatedUsing or Replicated tags. bReplicates must be set to true in the constructor.
```
	UPROPERTY(Replicated)
	bool RepVar = false;
 
	UPROPERTY(ReplicatedUsing = OnRep_ReadyForIMUSolveInit)
	bool RepVar2 = false;
```
Whenever the value of RepVar changes, the change propagates to clients. Whenever the value on RepVar2 changes, the change propagates to clients and OnRep_ReadyForIMUSolveInit function is called. 

```
void USomeComponent::GetLifetimeReplicatedProps(TArray<FLifetimeProperty>& OutLifetimeProps) const
{
	Super::GetLifetimeReplicatedProps(OutLifetimeProps);
	DOREPLIFETIME(USomeComponent, RepVar);
	DOREPLIFETIME(USomeComponent, RepVar2);
}
```
The above function must be overriden for proper replication of variables.
