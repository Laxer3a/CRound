# CRound
Attempt at creating a next gen C language specs.

Lately on Twitter a lot of debate is occuring about the complexity of "modern" C++.
I am wondering how we can take a language simple as C and make it better.

Note : for now I use the word struct as C struct + non virtual function possible inside.
Basically bit of object programming but not much. I have not decided how the OOP side of things should be handled.

## Axis 1 : Build and package.
- I like the idea that making a library can be released and downloaded into a single package.
- Namespace would determine the library usage and unicity of the name. (One namespace = one package)
  - Meta tag could define versions. (See Axis 2)
  - #import will replace #include and support optionnal version number if needed.
- No more parsing and include each time we compile a source => Package will have sources + build binary AST/content mendatory, optionnal prebuild obj for target platform.
  - If sources/AST only are provided, ability to build obj for a specific platform. => What about inline assembly ?
  - Sourceless (AST only) would be nice ?
- Generated .obj file are ELF. Compatible with existing linkers. Following ABIs etc...
- Can import C function statically (linking obj file from other compiler) or dynamically (dll like)

## Axis 2 : Reflection + meta data as default.
- Ability to list what is inside a project at execution time for a package in this language.
- Ability to put your own meta info on any item (function, struct, enum, etc...)
- Obviously you don't pay the price if you don't want to. (Features added/removed at linking time)
  
## Axis 3 : Remove mistakes/dangers from even C.
- Impossible to do assignment in conditionnal parameter : how many times people have been bitten by = instead of == inside a if / while / for loop.
  - Yes, you will write more code.
  - Yes, it will be less compact.
  - No, performance won't suffer : it is a compiler's job to fix that.
- Switch 'case' going through others are possible BUT EXPLICIT : through; <--- 'through' keyword is added to the language.
  - Again, avoid being bitten by forgetting break;
- Function and procedure are two different things (procedure = function returning void)
  - Ignoring the result of a function returning a value will require explicitely 'ignore' keyword.
- bool are not int. if (int) becomes if (int != 0) or (int == 0) like in Pascal or C#.
  - Therefore if/for/while condition are boolean.
- Cast are all explicit. Again you type more. Yes it is annoying when you are used to C, but leaves no implicit issue unverified by the programmer. (Same as C#)
  
## Axis 4 : Personal preferences.
- 'auto' like feature will not exist. If typing is too long, define a shortcut using typedef. Everything explicit.
   ex : thisPackage::struct::internalStruct  ==> typedef thisPackage::struct::internalStruct iStruct and voila !
- Add a 'nodefault' to 'default' possible switch cases : you garantee that the input you provide to the switch case is VALID for all cases (performance) and invalid values are UB (undefined behavior). (You own the responsability in exchange of performance gain explicit in the language. No more 'unreachable' compiler special magic)
- sizeof available at compile time for conditionnal macrolike setup.
- No exceptions.
- Memory allocation customization. Want the allocator to be replacable everywhere. Able to plug your own allocator in already existing libraries.
- Memory alignment is fist citizen too. Defined in the language. (member, struct)
- SIMD a first citizen (intrinsic support or inline assembly if you don't trust register allocators and optimizer from intrinsics)
- C has just pointers. C++ has reference added.
  - Reference is basically pointers again, except that NULL can not occur.
  - If reference supported, C# approach is very clean : in/out/ref usage is fully described BOTH at caller and callee. Forces initialization.
- Constant can use _ keyword to delimit and increase readability. ex : 0xDEAD_BE_EF
  - Binary constant are supported. ex : 0b0010_1100 (printf like formatting too)

## Axis 5 : Visibility
- public   : visible from everywhere.
- internal : visible from within the namespace.
- private  : visible from the struct itself only.

## Axis 6 : Boxed types
- All primitives types support object like stuff. (float.Biggest, float.Smallest, float.Parse, like C#)
  - Same memory size, no difference from compiler perspective.
  
## Axis 7 : Ability to extend class outside of the package.
- extends struct A { void myFunction(A* this, ...my parameters...); }
  - Useful to extend the string or float package. Ex : float.parse or string.ToUtf8() etc...
  - Only rule is that you are unable to access internals from the package and only the object as parameter.
  
## Axis 8 : .obj code generation ==> LLVM I think. Delegate the whole shenanigans to a well established backend.

## Undefined or have no idea yet
- Are macro a good thing ? C# seems too limiting. But would like to have the possibility for the compiler to execute the AST of any package and interpret this language. Allowing full C/C++ macro like stuff and manipulate the raw text seems 'too much'.
  - Have the ability for macro to define a block of code like a template is nice.
  - Have the ability to remove or select piece of code or macro is nice.
  - Replacing name with other name is evil (ex : #define true false )
- Inline assembler / ability to use assembler is a must. But what is the most convenient ? How to handle multiplatform as nicely as possible.
- Strings ? Seen as array would be nice. C string compatibility would be nice but how to do it cleanly. String manipulation too painful in C/C++. C++ string class too complex to my taste.
- Pointers, span and arrays.
  - Pointing to something and an array should be two different concepts. In one case you have only ONE item (like a reference), in the other case you delimit ranges and do some arithmetic,indexing on it.
- Templates ? Generics ?
  - It is nice to be able to write an algorithm with passing the type and avoid doing copy paste of code. I'd like to keep that.
  - Things like array, hashmap, etc.. Need some generic way of handling those.
  - But I want to avoid to make impossible template metaprogramming.
- memory fence/ordering/volatile
- multithreading / fibers
- GPU ? Future of heterogeneous computing ? Write code for different system within ? ==> Metatags can come handy.

## Standard libraries
- printf formatting is adapted to {1} number based, allowing to read the parameters orders the way you want. Other formatting parameters seems nice to keep.
  - Do we keep full compatibility for printf like stuff (porting all code)
