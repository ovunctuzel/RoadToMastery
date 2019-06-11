## C++ Study Journal

This page contains a list of quick tips and tutorials for programming and game development. This page mostly focuses on C++, but resources on C#, Python, Unreal Engine, Unity, and Math can also be found.
 
## Article List

### Programming

* [Endl is evil](#endl-is-evil-but-only-a-little-bit)
* [Data Alignment](#data-alignment)
* [Method Chaining](#method-chaining)
* [Using](#using)

### Unreal
* [Replication Tips](#replication-tips)

### Coming Soon
* Memset and friends
* Garbage collection
* Better exception handling
* Less known C++ data structures
* Implementation of C++ data structures
* Regex
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
