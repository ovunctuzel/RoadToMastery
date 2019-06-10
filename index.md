## C++ Study Journal

This page contains a list of quick tips and tutorials for programming and game development. This page mostly focuses on C++, but resources on C#, Python, Unreal Engine, Unity, and Math can also be found.
 
### Article List

* [Endl is evil](#endl-is-evil-but-only-a-little-bit)
* [Data Alignment](#data-alignment)
* [Method Chaining](#method-chaining)
* Memset and friends
* Garbage collection
* Better exception handling
* Less known C++ data structures
* Implementation of C++ data structures
* Regex
* Rvalue Lvalue
* Better Smart Pointers
* Better multithreading, std::atomic

## Endl is evil, but only a little bit
Std::endl is equivalent to ‘\n’ followed by a std::flush. Flushing is emptying the stream buffer, and it is not always desired. Not using std::endl everywhere can slightly improve performance.

## Data Alignment
In a C++ struct, data is padded with zeros to fit things in words. A struct with a char, an int, and a short takes up more space than a struct with an int, a short and a char. Only becomes relevant for millions of structs. More info: https://jonasdevlieghere.com/order-your-members/

## Method Chaining
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
