# C++ Cheatsheet
Quick lookup of arbitrary C++ rules

## Introduction
During the years I started to write very concise notes of C++ to quickly look up what I was missing. This became a very random set of cheatsheet entries, subjectively representing my learning. I intend this repo to have a permanent, formatted, structured storage for myself, but it can be used (and contributed) by anyone finding it.

## Pointer basics
- `*` -> *dereference* a.k.a. "pointed value". E.g. variable `a` is a pointer, `*a` is the value pointed at
- `*` is also declaring a variable that is meant to a pointer: `int *a` => `a` is a pointer to an `int`
- `&` *reference* i.e. "address of": `a = 54` -> `&a` is the address, where the value `54` is stored at

## Pass array reference to a function

In C++ you cannot pass an array to a function. However, there are three methods for passing an array by reference to a function:

- `void functionName(variableType *arrayName)`
- `void functionName(variableType arrayName[length of array])`
- `void functionName(variableType arrayName[])`

## Method pre- and postfixes

- `virtual`: the child classes method will be called even if you call it as a pointer to the base class. In case of a non-virtual method, the base classes method will be called. Mostly used to achieve runtime polymorphism.
- ` = 0` : a function is pure virtual and you cannot instantiate an object from this class. You need to derive from it and implement this method. You cannot declare a class abstract explicitly, but any class containing pure virtual methods are abstract. Only virtual functions can be pure virtual.
- ` = delete` : prohibiting calling (mostly used for disabling default behaviour, for example delete construction of singletons or delete the equal operator and copy constructor to prohibit copying)
    - Any use of a deleted function is ill-formed (the program will not compile).
    - You can prohibit some default behavior, for example normally you can copy a class even if you don't explicitly specify a copy constructor. By deleting the copy constructor you can explicitly prohibit copying.
- `= default` : you can mark a function explicitly to use the default, compiler implemented version. This is redundant, same as not writing the whole thing at all. This just helps you to see what's happening so you don't have to know the rules by heart.
- `explicit` : The explicit specifier specifies that a constructor or conversion function (since C++11) doesn't allow implicit conversions or copy-initialization.
const : this pointer becomes const. This function cannot modify the variables.
- `const` : the `this` pointer is const meaning the member function (method) cannot change the class members (prefer this if possible)

## `const` vs `constexpr`

- `constexpr`: we know the value *compile time*, compiler can substitute with literal
- `const`: might be decided *runtime*, but value is decided on assignment, cannot be changed later (actually can, e.g. with `const_cast`)

Rule of thumb: for performance reasons, use constexpr if possible

## 

## `std::bind`

Creates a new std function with "predefined" arguments.
	
Example: let's assume we have a function called `f`, with 1 integer arugment. `std::bind(f, 3)` returns a function. The returned function has no arguments. If you call this function, it will call `f(3)`.
	
Placeholders:
	
`f1 = std::bind(f, 3, _1)`, you should call `f1(something)` and something will be substituted to `_1`, therefore `f(3, something)` will be called.
	
## Instantiate an object

```cpp
Entity entity; // Calls the default constructor (not null or something)

Entity entity = Entity("something") // Calls the specified constructor

Entity entity("somethign") // Same as above, just shorter in code

Entity* entity = new Entity("somethign") // Heap allocation, slower and manually  needs to be deleted, but survives the scope's end

Entity entity{"something"} // Called "uniform initialization", can be used the same way all over, e.g. int:

int a{5}; // equals to int a = 5;

// Smart pointers (see their own sectin)
// TODO
```

See more about uniform initialization here: https://www.geeksforgeeks.org/uniform-initialization-in-c/

### Some notes abour instantiation

Thing about performance. If you create an object, and initialize it later, a copy or reassignment can happen. Do it in 1 step. (Similar applies for initializer lists. They are better to be used, because setting members inside the constructor would mean duoble work: creating the object with default values, then reassign those values, while initializer list creates the object with the given values in the first place)

## casts

- `static_cast`: most common, compile time safe casts (e.g. child class to base class). IDE can check usually
- `dynamic_cast`: mostly base to child, performs runtime check, returns `nullptr` if it wasn't successful. Can be used to check type of variable. (Similar cases when you would use something like )
- `const_cast`: modifies the constness of the target
- `reinterpret_cast`: converts pointer to different type without checking or modifying the pointed at data. Just starts to interpret the same data as a different type wihtout any considerations (type punning)

## Rules of thumb

There are some basic rules to improve performance. These can always be broken, so use them with understanding the whys and always consider alternatives.

### `#include` order

`#include` directives should go from local to global.

It doesn't just add consistency, but helps to discover bugs that would hide otherwise. If your local header misses an `#include` directive, the problem migth be shadowed if your client code includes it before the header. Your code might compile, but 1) if you change your `#include`s later, or 2) you include your header somewhere else, the compile error might come up. Using this rule can surface such problems during development:

1. In case of a `.cpp` file, first include is it's own header
2. Other local headers of your project
3. Third party headers
4. System headers

In general, start as local as possible, and gradually go further and bigger and more global.

### Variables and values

- If a value can be `constexpr`, it should be. If not, but can be `const`, it should be. Use variables if not.
- In general, everything that can be `const`, should be `const` (see details later), e.g. function parameters, class methods, iterators, etc.

### Funtion and method parameters

- With function and method parameters, everything apart from the smalles PODs should be passed as reference, otherwise they will be copied by value which is slow.
- With function and method parameters, everything should be const (especially those passed by reference) if there is no explicit reason for the contrary.
- Putting these 2 together, most of the function parameters will be "const refs", like `f(const std::string& input);` instead of `f(std::string input)`

### Class members

- Everything should be private unless there is explicit reason to expose it (avoid "just in case" public members)
- Generally member variables should be private and be accessed by getters and setters. Might be reasoned otherwise (even if you don't need getters and setters now, but might be introduced later, you can do it wihtout API break if you start using them from the beginning)

### Class methods

- Every class method that doesn't use members should be placed outside the class. If there is still reason to include it in the class, it should be static. It helps a lot to the compiler and to you when debugging.
- Every class methos that reads, but doesn't write members, should be marked as `const`

## Questions

### Question - how to link

There is a main program, that includes a header from my project (MyClass.h). The header has a cpp file with the implementation (MyClass.cpp), and also includes the MyClass.h. The cpp files, according to the practice, don't refer to each other. Otherwise these files are structured in the common way, i.e. the header contains declarations, and the cpp specifies the implementations, for example for the class methods.

So how does the linker know, how to find the implementations? Let's say I call myClass->myFunction(), which is declared in the MyClass.h, but implemented in the MyClass.cpp.

First, the header is kind of in the "middle". The client code and it's own implementation are exactly the same from the include structure perspective.

My understanding: from the client code, including the "MyClass.h" provides (forward) declarations, that are enough to satisfy the compiler, but obviously not enough runtime.

Naively I would think, when the compiler runs, it starts witht he entry point, and tries to follow the includes, like "okay, I compile this single file. AHa, so this is included, then I need to compile this too", etc. This approach wouldn't work, as the implementations are never included, so they are not part of a dependency tree built this way.

So my understanding is, the compiler is rather compiling all the files within the project, or whatever is set in the compiler's configurations, point is, it is compiling a list of files regardless of their include relations.

Now that I think of it, it makes sense, as includes are not visible for the compiler, they are already resolved by the preprocessor at compile time.

So if we put everything together nicely, we will have usage(s) (cliend code), declaration (h) and definition(cpp) of each function, just like we would have them in one single file. What connects them is the signature. There can be only one declaration and also definition of a function signature.

Self test:
- So can we have a class declaration in the header, and provide implementation in different files? Should be, if I am correct.
- If both includes the hearder, what if I only write 2 cpp files both declaring the same functions, without referring to each otherÜ

### Question - include header dependencies from the cpp

All trainings say includes are just copying text to this file from another.

Let's say I create a header file, called MyTest.h. If I use a class (Foo) in it, without including of forward declaring anything, it doesn't compile. It makes sense.

Now I create a corresponding cpp file, called MyTest.cpp, that includes MyTest.h. Then, if I include the above class (let's say "Foo.h") in the .cpp, the header starts to compile. The .cpp includes the .h, but the .h doesn't have any reference to the .cpp whatsoever. Header should be independent from the cpp. How do we know that that cpp file will be used for the header?

Also, when I rename them (they have different names even apart from the file extension, like "MyTest.h" and "OtherFile.cpp"), suddenly the .h file stops compiling. It contradits the fact that these are all conventions, and even .h or .cpp doesn't have a specific meaning to the compiler. I assume, maybe it's some additional Visual Studio logic, but it still shouldn't affect what to include where.

Apparently, if I have more than 1 files including the header file, it stops compiling again. This, again, makes sense. So the case is, if I have 1 single file including the header, AND that is named the same way as the header, then it's enough to include dependencies in the cpp. 

So how it is?

