Introduction
=

Jai is a high-level programming language developed by [Jonathan Blow](https://twitter.com/Jonathan_Blow), creator of indie games Braid and any-time-now-to-be-released The Witness. It is an imperative static/strongly typed C-style language, but with a variety of modern language features that C lacks. Blow began work on Jai in late September 2014. It is still in development and as of yet is unavailable to the general public. Blow developed it with an eye towards video games, but in fact it’s a general purpose programming language that could be used for any task.

DISCLAIMER: I have no association with Jon Blow. As of this writing there are no public compilers for Jai, so all information in this text is collated from [Jon Blow's YouTube videos](https://www.youtube.com/playlist?list=PLmV5I2fxaiCKfxMBrNsU1kgKJXD3PkyxO). Therefore nothing in this post is official. There may be information more up to date than what is available on this page. That said, I believe everything in this post to be up to date as of this writing. (If you are Jon Blow and want me to correct anything in this post, [I would be happy to.](http://twitter.com/VinoBS))

Everything in this document, unless otherwise noted, is implemented and currently working in Blow's private prototype, but as it is not yet released, everything is subject to change.

Brief Description
-

In short, Jai could be described as a modern replacement for C. Some of the coolest features:

- Arbitrary compile-time code execution - Any function of the program can be made to run at compile time with #run
- Syntax-facilitated code refactoring - The language syntax facilitates code reuse by making it easy to move code from local block → local function → global function
- Integrated build process - The build process and parameters are specified by the source code itself, for consistency
- Data-oriented structures - Automatic conversion between Structure of Arrays and - Array of Structures, avoids classes and inheritance
- Reflection and run-time type information - Static type information for every structure available at runtime
- A new approach to polymorphic procedures - Polymorphism at the function level, with programmer control by special procedures
- Low-level memory management tools - Better control over how libraries allocate memory, automatic ownership management, no garbage collection
- Explicit control over optimization and performance characteristics - Explicit control over things like inlining, bounds checking, and initialization

The Philosophy of Jai
=

**THE JOY OF PROGRAMMING**

At some point after programming for many years, the line between “exciting programming adventure” and “please not another code refactoring” can start to disappear. Having to update the function declaration in the header when you change its signature gets old fast. When C was originally written in 1973, there was a good reason for all that header stuff, but today there isn’t. Quality of life improvements afforded by the language can have quantifiable benefits to the productivity of the programmer using the language. (If you aren’t convinced of that, try programming anything in [Brainfuck](http://en.wikipedia.org/wiki/Brainfuck).) Compiling should be fast if not instantaneous, refactoring code should require a minimum of changes, and error messages should be helpful and pleasant. Blow believes that an improvement of the tools that programmers use can produce more than a 20% increase in productivity, and this was a primary motivation in creating a new language.

**MACHINES THAT FILL MEMORY**

Video games are, as Blow puts it, machines that fill memory. The majority of the time, game programmers are thinking about how to fill memory with huge reams of data in ways that allow the data to be efficiently accessed and processed. Hundreds of megabytes of memory must be moved from the hard disk into main memory, and from there into the video card or the processor cache to be processed and returned back to main memory. Because video game players don't like to wait, all this must be done as fast as is allowable by laws of our universe. The primary purpose of a programming language is to allow the specification of algorithms to manage data. Language features like garbage collection and templated data streams and dynamic string classes may help the programmer write code faster, but they don't help the programmer write faster code.

**FRICTION REDUCTION**

Another major design goal of Jai is to reduce what Blow calls friction in programming. Friction happens when the syntax of a language interferes with the programmer's workflow. When Java requires that all objects be classes, forcing programmers to put the global variables they need into global classes, that's friction. When Haskell requires that all procedures be functions and have no side effects, that's friction. When C++'s lambda function syntax is different from its class method syntax is different from its global function syntax, that's friction. Java Haskell and C++ are examples of what Blow calls "big agenda" languages, where the idealism (and in C++’s case, lack of a consistent vision) of the language gets in the programmer’s way. Blow has a low tolerance for friction in his language, especially when the friction is unnecessary.

**DESIGN FOR GOOD PROGRAMMERS**

Blow wants a language that is designed for good programmers, not against bad programmers. Languages like Java were marketed as idiot-proof, in that it’s much more difficult for programmers to write code that can hurt them. The Jai philosophy is, if you don’t want idiots writing bad code for your project, then don’t hire any idiots. Jai allows programmers direct access to the sharp tools that can get the job done. Game programmers are not afraid of pointers and manual memory management. Programmers do make mistakes and cause crashes, perhaps even serious ones, but Blow argues that the increase in productivity and reduction of friction when memory-safe mechanisms are absent more than make up for the time lost in tracking down errors, especially when good programmers tend to produce relatively few errors.

**PERFORMANCE AND DATA-ORIENTED PROGRAMMING**

If as a programmer you care about user experience, (which you should) then you should care about the performance of your program. You should reason about your code's behavior on the range of machines that you're shipping on, and design your data and control structures to use that hardware's capability most efficiently. (Here I'm describing [Mike Acton's "Data-Oriented Design" methodology](https://www.youtube.com/watch?v=rX0ItVEVjHc).) Programmers who care about the performance of their software on their target hardware are inhibited by programming languages that sit between them and the hardware. Mechanisms like virtual machines and automatic memory management interfere with the programmer’s ability to reason about the program’s performance on the target hardware. Abstractions like RAII, constructors and destructors, polymorphism, and exceptions were invented with the intention of solving problems that game programmers don’t have, and with the result of interfering with the solutions to problems that game programmers do have. Jai jettisons these abstractions so that programmers can think more about their actual problems - the data and their algorithms.

Jai Language Features
=

Types and Declarations
-

The syntax `name: type = value;` specifies that a variable named name is of the type type and is to receive the value value. It was proposed by [Sean Barrett](https://twitter.com/nothings). Some examples:

```cpp
counter: int = 0;
name: string = "Jon";
average: float = 0.5 * (x+y);
```

If the type is omitted then the compiler infers it based on the value:

```cpp
counter := 0;           // an int
name := "Jon";          // a string
average := 0.5 * (x+y); // a float
```

If the value is omitted then you have a declaration without an initialization.

```cpp
counter: int;
name: string;
average: float;
```

All of this is probably backwards from what you’re used to, but the learning curve is shallow and you get used to it quickly. Function declarations look like this:

```cpp
// A function that accepts 3 floats as parameters and returns a float
sum := (x: float, y: float, z: float) -> float {
    return x + y + z;
};

print("Sum: %\n", sum(1, 2, 3));
```

and structure declarations like this:

```cpp
Vector3 := struct {
    float x;
    float y;
    float z;
};
```

Arrays can be created like this:

```cpp
a: [50] int; // An array of 50 integers
b: [..] int; // A dynamic array of integers
```

Arrays do not automatically cast to pointers as in C. Rather, they are "wide pointers" that contain array size information. Functions can take array types and query for the size of the array.

```cpp
print_int_array :: (a: [] int) {
    n := a.count;
    for i : 0..n-1 {
        print("array[%] = %\n", i, a[i]);
    }
}
```

Retaining the array size information can help developers avoid the pattern of passing array lengths as additional parameters and assist in automatic bounds checking (see Walter Bright - C's Biggest Mistake)
Arbitrary Compile-Time Code Execution
-

Suppose I want to write a function in C that converts a linear color value to [sRGB](http://en.wikipedia.org/wiki/SRGB). This involves the pow() function, which is on the expensive side. We can avoid pow() by doing the calculation ourselves instead and distributing the results as part of our program. So we write a table of values and return those.

```cpp
#define SRGB_TABLE_SIZE 256
float srgb_table[SRGB_TABLE_SIZE] = { /* ... values here ... */ }

float linear_to_srgb(float f)
{
    // Find the index in our table for this SRGB value,
    // assuming f is in the range [0, 1]
    int table_index = (int)(f * SRGB_TABLE_SIZE);
    return srgb_table[table_index];
}
```

(Note: The above is bad code, only used for example. For better code, try [stb_image_resize’s sRGB functions](https://github.com/nothings/stb/blob/master/stb_image_resize.h).) So far so good, except how will we get the values for the srgb_table? We can write another small program that outputs values. For example:

```cpp
float real_linear_to_srgb(float f)
{
    if (f <= 0.0031308f)
        return f * 12.92f;
    else
        return 1.055f * (float)pow(f, 1 / 2.4f) - 0.055f;
}

#define SRGB_TABLE_SIZE 256

int main(int c, char* s) {
    printf("float srgb_table[SRGB_TABLE_SIZE] = { ");
    for (int i = 0; i < SRGB_TABLE_SIZE; i++)
        printf("%f, ", real_linear_to_srgb((float)i/SRGB_TABLE_SIZE));
    printf("}\n");
    return 0;
}
```

We can compile this small program, which will output a table of sRGB values, and then we can copy the output into our actual program.

This is a big bucket of problems with it. For example, notice how SRGB_TABLE_SIZE is defined twice, once in the actual program and once in the helper program. So we now have to maintain two separate source codes. This can get unwieldy for large programs.

In Jai, the same task looks like this:

```cpp
generate_linear_srgb := () -> [] float {
     srgb_table: float[SRGB_TABLE_SIZE];
     for srgb_table {
         << it = real_linear_to_srgb(cast(float)it_index / SRGB_TABLE_SIZE)
     }
     return srgb_table;
}

srgb_table: [] float = #run generate_linear_srgb(); // #run invokes the compile time execution

real_linear_to_srgb := (f: float) -> float {
    table_index := cast(int)(f * SRGB_TABLE_SIZE);
    return srgb_table[table_index];
}
```

The #run directive instructs Jai to run the function generate_linear_srgb() at compile time. Jai’s compile time function execution runs the command at compile time and returns a table of values, which is then compiled directly into the binary as srgb_table. When the program is run, the generate_linear_srgb() function no longer exists. Only the table it generated exists, which is used by linear_to_srgb().

The compile-time function execution has very few limitations, in fact you can run arbitrary code in your code base as part of the compiler. In Blow’s first demonstration he shows how to run [an entire game as part of the compiler](http://youtu.be/UTqZNujQOlA?t=43m57s), and bake the data from the game into the program binary. (I hope `#run invaders();` is shipped with the language.) The compiler builds the compile-time executed functions to a special byte code language and runs them in an interpreter, and the results are funneled back into the source code. The compiler then continues as normal.

Here are some examples of things that a compile-time function could do:

Compile-time asserts

- List item
- Run test cases
- Do code style checks
- Dynamically generate code and insert it to be compiled
- Insert build time data
- Download the OpenGL spec and build the most recent gl.h header file
- Contact a build server and retrieve/send build data
- Talk to your Mars probe on Mars and wait for the packets to come back and get a photo of what Mars looks like

Code Refactoring
-

All code begins its life in some kind of code block like this before moving on to be used in more general cases. Jai has some special syntaxes that can assist the programmer in moving code from specific cases out into general cases, to facilitate code reuse.

As an example, let’s say you’re writing some code like this:

```cpp
draw_particles := () {
    view_left: Vector3 = get_view_left();
    view_up: Vector3 = get_view_up();

    for particles {
        // Inside for loops the "it" object is the iterator for the current object.
        particle_left := view_left * it.particle_size;
        particle_up := view_up * it.particle_size;

        // m is a global object that helps us build meshes to send to the graphics API
        m.Position3fv(it.origin - particle_left - particle_up);
        m.Position3fv(it.origin + particle_left - particle_up);
        m.Position3fv(it.origin + particle_left + particle_up);
        m.Position3fv(it.origin - particle_left + particle_up);
    }
}
```

These mesh generation calls are actually a special case of some general quad rendering, so they can be factored out into another function so they can be used in other places. Jai makes this refactoring very straightforward. The first step is to enclose the code in a new scope with a special capture syntax.

```cpp
particle_left := view_left * it.particle_size;
particle_up := view_up * it.particle_size;
origin := it.origin;

[m, origin, particle_left, particle_up] {
    m.Position3fv(origin - particle_left - particle_up);
    m.Position3fv(origin + particle_left - particle_up);
    m.Position3fv(origin + particle_left + particle_up);
    m.Position3fv(origin - particle_left + particle_up);
}
```

(Disclaimer: This step hasn't been implemented by Blow yet. It's one of his planned features.) The `[m, origin, particle_left, particle_up]` notation is a capture that prevents any object not in the capture from being accessed inside the inner scope of the new bracket. Notice that we had to change “it.origin” to “origin” and add `origin` to the capture list -- “it” is not captured and is unavailable inside the inner scope.

Captures help in refactoring code as we’re seeing here but they can also help in other ways. For example, when programmers are moving code from being singlethreaded to multithreaded, captures could enforce that only thread-local data is accessed. Captures are an insurance policy that the code inside the capture only reads or writes the state specified in the capture.

Now we’ve identified all of the parts of our code that depend on external things, so we’ve improved our code’s hygiene and made it easy to pull this code out into its own function. Now we want to continue so that we can use the quad drawing code in other places. So we create a function out of this block capture:


```cpp
particle_left := view_left * it.particle_size;
particle_up := view_up * it.particle_size;
origin := it.origin;

() [m, origin, particle_left, particle_up] {
    m.Position3fv(origin - particle_left - particle_up);
    m.Position3fv(origin + particle_left - particle_up);
    m.Position3fv(origin + particle_left + particle_up);
    m.Position3fv(origin - particle_left + particle_up);
} (); // Call the function
```

Notice how the only change we needed to make was to add the function syntax (). The capture remained intact. So we went from a blocked capture to a function with very little effort. Now if we like we can move the vectors to be function parameters:

```cpp
(origin: Vector3, left: Vector3, up: Vector3) [m] {
    m.Position3fv(origin - left - up);
    m.Position3fv(origin + left - up);
    m.Position3fv(origin + left + up);
    m.Position3fv(origin - left + up);
}
```

With parameter names we’re able to change the names of the variables inside the function’s scope to match their new function. Now we can use this function to draw any type of quad, not just particles. The capture retains m because m is a global object that doesn’t need to be passed as a parameter. And now we have an anonymous, locally scoped function that can be used in our draw code:

```cpp
draw_particles := () {
    view_left: Vector3 = get_view_left();
    view_up: Vector3 = get_view_up();

    for particles {
        particle_left := view_left * it.particle_size;
        particle_up := view_up * it.particle_size;

        (origin: Vector3, left: Vector3, up: Vector3) [m] {
            m.Position3fv(origin - left - up);
            m.Position3fv(origin + left - up);
            m.Position3fv(origin + left + up);
            m.Position3fv(origin - left + up);
        } (origin, particle_left, particle_up);  // Call the function with the specified parameters
    }
}
```

Anonymous functions are useful for passing as arguments to other functions, and this syntax makes them easy to create and manipulate. The next step is to give our function a name:

```cpp
draw_quad := (origin: Vector3, left: Vector3, up: Vector3) [m] {
    m.Position3fv(origin - left - up);
    m.Position3fv(origin + left - up);
    m.Position3fv(origin + left + up);
    m.Position3fv(origin - left + up);
}

draw_quad(origin, particle_left, particle_up);
```

Now we could call it multiple times in the local scope, if we like. But we want to access our quad drawing function from the global scope. Moving the function out of the local scope requires zero changes to the function’s code:

```cpp
draw_quad := (origin: Vector3, left: Vector3, up: Vector3) [m] {
    m.Position3fv(origin - left - up);
    m.Position3fv(origin + left - up);
    m.Position3fv(origin + left + up);
    m.Position3fv(origin - left + up);
};

draw_particles := () {
    view_left: Vector3 = get_view_left();
    view_up: Vector3 = get_view_up();

    for particles {
        particle_left:= view_left * it.particle_size;
        particle_up:= view_up * it.particle_size;

        draw_quad(particle_left, particle_up, origin);
    }
}
```

The strength of Jai’s function syntax is that it doesn’t change whether the function is an anonymous function, a local function (i.e. lives inside the scope of another function) a member function of a class or a global function. This is in contrast to in C++, where a local function is called a lambda, and has completely different syntax than a member function, which must have a class name and `::` and so on, which is slightly different syntax than a global function which has no class name or `::`. The result is that as code matures and moves from a local context to a global context, the work of refactoring can be done with minimal edits.

Here is Jai’s the code maturation cycle in full:

```cpp
                                 { ... } // Anonymous code block
                       [capture] { ... } // Captured code block
     (i: int) -> float [capture] { ... } // Anonymous function
f := (i: int) -> float [capture] { ... } // Named local function
f := (i: int) -> float [capture] { ... } // Named global function
```

Integrated Build Process
-

All information for building a program is contained within the source code of the program. Thus there is no need for a "make" command or project files to build a Jai program. As a simple example:

```cpp
build :: () {
    build_options.executable_name = "my_program";
    print("Building program '%'\n", build_options.executable_name);
    build_options.optimization_level = Optimization_Level.DEBUG;
    build_options.emit_line_directives = false;

    update_build_options();

    // Jai will automatically build any files included with the #load directive, but other files can also be manually added
    add_build_file("misc.jai");
    add_build_file("checks.jai");
}

#run build();
```

When the program is built, the #run directive runs build() at compile-time. Then build() establishes all of the build options for this project. No external build tools are required, all build scripting is done within Jai, and in the same environment of the rest of the code.
Data-Oriented Structures
-

**SOA AND AOS**


Modern processors and memory models are much faster when spatial locality is adhered to. This means that grouping together data that is modified at the same time is advantageous for performance. So changing a struct from an array of structures (AoS) style:

```cpp
struct Entity {
    Vector3 position;
    Quaternion orientation;
    // ... many other members here
};

Entity all_entities[1024]; // An array of structures

for (int k = 0; k < 1024; k++)
    update_position(&all_entities[k].position);

for (int k = 0; k < 1024; k++)
    update_orientation(&all_entities[k].orientation);
```

to a structure of arrays (SoA) style:

```cpp
struct Entity {
    Vector3 positions[1024];
    Quaternion orientations[1024];
    // ... many other members here
};

Entity all_entities; // A structure of arrays

for (int k = 0; k < 1024; k++)
    update_position(&all_entities.positions[k]);

for (int k = 0; k < 1024; k++)
    update_orientation(&all_entities.orientations[k]);
```

can improve performance a great deal because of fewer cache misses.

However, as programs get larger, it becomes much more difficult to reorganize the data. Testing whether a single, simple change has any effect on performance can take the developer a long time, because once the data structures must change, all of the code that acts on that data structure breaks. So Jai provides mechanisms for automatically transitioning between SoA and AoS without breaking the supporting code. For example:

```cpp
Vector3 :: struct {
    x: float = 1;
    y: float = 4;
    z: float = 9;
}

v1 : [4] Vector3;     // Memory will contain: 1 4 9 1 4 9 1 4 9 1 4 9

Vector3SOA :: struct SOA {
    x: float = 1;
    y: float = 4;
    z: float = 9;
}

v2 : [4] Vector3SOA;  // Memory will contain: 1 1 1 1 4 4 4 4 9 9 9 9
```

Getting back to our previous example, in Jai:

```cpp
Entity :: struct SOA {
    position : Vector3;
    orientation : Quaternion
    // .. many other members here
}

all_entities : [4] Entity;

for k : 0..all_entities.count-1
    update_position(&all_entities[k].position);

for k : 0..all_entities.count-1
    update_orientation(&all_entities[k].orientation);
```

Now the only thing that needs to be changed to convert between SoA and AoS is to insert or remove the SOA keyword at the struct definition site, and Jai will work behind the scenes to make everything else work as expected.
Reflection and Run-Time Type Information

Jai stores a table of all type information in the data segment of each compiled program. It can be examined like this:

```cpp
for _type_table {
    // it is the iterator, it is the Type being examined. it_index is the iteration index, it is an integer
    print("%:\n", it_index);
    print("  name: %\n", it.name);
    print("  type: %\n", it.type); // type is an enum, INTEGER, FLOAT, BOOL, STRUCT, etc
}
```

Full introspection data is available for all structs, functions, and enums. For example, a procedure may look something like this:

```cpp
print("% (", info_procedure.name);
for info_procedure.argument_types {
    print_type(it);
    if it_index != info_procedure.argument_types.count-1 then print(", ");
}
print(") ->");
print_type(info_procedure.return_type);
```

The preceding code could print something like, `get_name(id : uint32) -> string`. An enum can be examined like this:

```cpp
Hello :: enum u16 {
    FIRST,
    SECOND,
    THIRD = 80,
    FOURTH,
}

for Hello.names {
    print("Name: % value: %\n", Hello.names[it_index], Hello.values[it_index]);
}
```

Reflection data such as this can be used to write serialization procedures, commonly used e.g. in network replication of entities and save game data. Current C/C++ methods for this involve heavy use of operator overloading and preprocessor directives.
Polymorphic Procedures
-

**FUNCTION POLYMORPHISM**

Jai's primary polymorphism mechanism is at the function level, and is best described with an example.

```cpp
sum(a: $T, b: T) -> T {
    return a + b;
}

f1: float = 1;
f2: float = 2;
f3 := sum(f1, f2);

i1: int = 1;
i2: int = 2;
i3 := sum(i1, i2);

x := sum(f1, i1);

print("% % %\n", f3, i3, x); // Output is "3.000000 3 2.000000"
```

When sum() is called, the type T is determined by the T which is preceded by the \$ symbol. In this case, the \$ symbol precedes the a variable, and so the type of T is determined by the first parameter. So the first call to sum() is float + float, and the second call is int + int. In the third call, since the first parameter is float, both parameters and the return value become float. The second parameter is converted from int to float, and the variable x is deduced to be float as well.

**THE ANY TYPE**

Jai has a type called Any, which any other type can be implicitly casted to. Example:

```cpp
print_any(a: Any) {
    if a.type.type == Type_Info_Tag.FLOAT
        print("a is a float\n");
    else if a.type.type == Type_Info_Tag.INT
        print("a is an int\n");
}
```

**BAKING**

... this section is not written yet! Sorry. (The #bake directive emits a function with a combination of arguments baked in. e.g. `#bake sum(a, 1)` becomes equivalent to `a += 1`.)
Memory Management
-

Jai does not and will never feature garbage collection or any kind of automatic memory management.

**STRUCT POINTER OWNERSHIP**

Marking a pointer member of a struct with ! indicates that the object pointed to is owned by the struct and should be deleted when the struct is deallocated. Example:

```cpp
node := struct {
    owned_a : node *! = null;
    owned_b : node *! = null;
};

example: node = new node;
example.owned_a = new node;
example.owned_b = new node;

delete example; // owned_a and owned_b are also deleted.
```

Here, owned_a and owned_b are marked as being owned by node, and will be automatically deleted when the node is deleted. In C++ this is accomplished through a unique_ptr<T>, but Blow thinks that this is the wrong way to do it because the template approach now masks the true type of the object. A unique_ptr<node> is no longer a node, it’s a unique_ptr masquerading as a node. It’s preferable to retain the type of node*, and retain the properties of node*-ness that go along with it, because we don’t really actually care about unique_ptr for its own sake.

**LIBRARY ALLOCATORS**

... this section is not written yet! Sorry. (Jai provides mechanisms for managing the allocations of an imported library without requiring work from the library writers.)
Explicit Performance Control

**INITIALIZATION**

Member variables of a class are automatically initialized.

```cpp
Vector3 :: struct {
    x: float;
    y: float;
    z: float;
}

v : Vector3;
print("% % %\n", v.x, v.y, v.z); // Always prints "0 0 0"
```

You can replace these with default initializations:

```cpp
Vector3 :: struct {
    x: float = 1;
    y: float = 4;
    z: float = 9;
}

v : Vector3;
print("% % %\n", v.x, v.y, v.z); // Always prints "1 4 9"

va : [100] Vector3;                                 // An array of 100 Vector3
print("% % %\n", va[50].x, va[50].y, va[50].z); // Always prints "1 4 9"
```

Or you can block the default initialization:

```cpp
Vector3 :: struct {
    x: float = ---;
    y: float = ---;
    z: float = ---;
}

v : Vector3;
print("% % %\n", v.x, v.y, v.z); // Undefined behavior, could print anything
```

You can also block default initialization at the variable declaration site:

```cpp
Vector3 :: struct {
    x: float = 1;
    y: float = 4;
    z: float = 9;
}

v : Vector3 = ---;
print("% % %\n", v.x, v.y, v.z); // Undefined behavior, could print anything

va : [100] Vector3 = ---;
print("% % %\n", va[50].x, va[50].y, va[50].z); // Undefined behavior, could print anything
```

By explicitly uninitializing variables rather than explicitly initializing variables, Jai hopes to reduce cognitive load while retaining the potential for optimization.

**INLINING**

```cpp
test_a :: () { /* ... */ }
test_b :: () inline { /* ... */ }
test_c :: () no_inline { /* ... */ }

test_a(); // Compiler decides whether to inline this
test_b(); // Always inlined due to "inline" above
test_c(); // Never inlined due to "no_inline" above

inline test_a(); // Always inlined
inline test_b(); // Always inlined
inline test_c(); // Always inlined

no_inline test_a(); // Never inlined
no_inline test_b(); // Never inlined
no_inline test_c(); // Never inlined
```

Additionally, there exist directives to always or never inline certain procedures, to make it easier to inline or avoid inline conditionally depending on the platform.

```cpp
test_d :: () { /* ... */ }
test_e :: () { /* ... */ }

#inline test_d    // Directive to always inline test_d
#no_inline test_e // Directive to never inline test_d
```


Other Cool Stuff
-
Things that C/C++ should have had a long time ago:

- Multi-line block comments
- Nested block comments
- Specific data types for 8, 16, and 32 bit integers
- No implicit type conversions
- No header files
- . operator for both struct membership and pointer dereference -- no more ->
- A defer statement, [similar to that of Go](http://blog.golang.org/defer-panic-and-recover).

Planned
-

Here’s a short list of features that Blow has expressed interest in for Jai.

- Automatic build management -- the program specifies how to build it
- Captures
- LLVM integration
- Automatic versioning (see below)
- A better concurrency model
- Named argument passing
- A permissive license

Not Planned
-

Jai will not have:

- Smart pointers
- Garbage collection
- Automatic memory management of any kind
- Templates or Template Meta-Programming
- RAII
- Constructors and Destructors
- Polymorphism
- Exceptions
- References
- A virtual machine (at least, not usually - see below)
- A preprocessor (at least, not one resembling C's - see below)
- Header files

If it sounds odd to you that Jai is a modern high-level language but does not have some of the above features, then consider that Jai is not trying to be as high-level as Java or C#. It is better described as trying to be a better C. It wants to allow programmers to get as low-level as they desire. Features like garbage collection and exceptions stand as obstacles to low-level programming.
Further Notes
=

**ADOPTION**

A compelling argument for not writing an entirely new language for games is that the momentum and volume of C and C++ code in current game engines is too high, and switching to a new language is too much work for the amount of benefit. Blow argues that engines periodically rewrite their codebase anyway, and since Jai and C are so closely related, C code and Jai code can live side by side while the rewrites that would normally happen anyway take place. Since C and Jai interoperate seamlessly, Jai code can be built on top of existing C libraries. In fact, Blow uses the C interfaces to the OpenGL and [stb_image](https://github.com/nothings/stb/blob/master/stb_image.h) libraries for his Jai test code. So, replacing C and C++ can be done with no added cost to development. Meanwhile, the benefits of replacing C with a language that has all of C’s benefits but fewer drawbacks means that programmers will be happier, and thus more productive.
Why not use ... ?
-

**WHY NOT USE C++/RUST/GO/D/SWIFT/HASKELL/LISP/ETC?**

Those are strong languages, but none of them contain the right combination of features (or lack of features) that game programmers need. Automatic memory management is a non-starter for game programmers who need direct control over their memory layouts. Any interpreted language will be too slow. Functional-only languages are pointlessly restricting. Object-oriented-only languages are overly complex. Blow preferred to develop a new language with the qualities that game programmers need, and without the qualities they don’t.
Proposed Features
-

These are a few features that Blow has proposed but not yet implemented. To my knowledge they’re not yet in the language. Syntax is preliminary and likely to change.

The first is joint allocations:

```cpp
Mesh :: struct {
    name: string;
    filename: string;
    flags: uint;

    positions: [] Vector3;
    indices: [] int;        @joint positions
    uvs: [] Vector2;        @joint positions
};

example_mesh: Mesh;
example_mesh.positions.reserve(positions: num_positions,
                               indices: num_indices,
                               uvs: num_uvs);
```

Here we want to avoid multiple memory allocations, so we have the compiler do a joint allocation between positions, indices, and uvs, and divide the memory up accordingly. Currently this is done manually in C, and is prone to errors.

Next is optional types:

```cpp
do_something := (a: Entity?) {
    a.x = 0; // ERROR!
    if (a) {
        a.x = 0; // OK
    }
};
```

The idea here is to prevent one of the most common causes of crashes, null pointer dereferencing. The ? in the above code means that we don’t know whether or not the pointer is null. Trying to dereference the variable without testing to see if it is null would result in a compile time error.

Lastly, automatic versioning:

```cpp
Entity_Substance :: struct { @version 37
    flags: int;
    scale_color: Vector4;   @15
    spike_flags: int;       @[10, 36]
};
```

Here Blow is providing markup for his data structures that indicates to the compiler what version of the struct each member was present in. These versioning schemes would be used as part of an automatic serialization/introspection interface, but he’s not gone into details on that other than that the language should have some capability of introspection.
