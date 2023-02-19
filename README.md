# CRound
Attempt at creating a next gen C language specs.

Lately on Twitter a lot of debate is occuring about the complexity of "modern" C++.
I am wondering how we can take a language simple as C as a base, modify it and try to make it better.

Note : for now I use the word struct as C struct + non virtual function possible inside.
Basically bit of object programming but not much. I have not decided how the OOP side of things should be handled.

## Idea 1 : Build and package.
- I like the idea that making a library can be released and downloaded into a single package (single file binary file, not an archive !!!).
  - Provide tool to split package again into sources if provided inside package.
- Namespace would determine the library usage and unicity of the name. (One namespace = one package)
  - Meta tag could define versions. (See Idea 2)
  - #import will replace #include and support optionnal version number if needed.
- No more parsing and include each time we compile a source => Package will have sources + build binary AST/content mendatory, optionnal prebuild obj for target platform.
  - If sources/AST only are provided, ability to build obj for a specific platform. => What about inline assembly ?
  - Sourceless (AST only) would be nice ?
- Cargo idea is nice in Rust. (Unit test/validation, Benchmark, connection to a DB online checking downloading/uploading versions, etc...)

## Idea 1' : Outside world.
- Generated .obj files are ELF. Compatible with existing linkers. Following ABIs etc... Basically export C functions.
- Can import C function statically (linking obj file from other compiler) or dynamically (dll like)

## Idea 2 : Reflection + meta data as default.
- Ability to list what is inside a project at execution time for a package in this language.
- Ability to put your own meta info on any item (function, struct, enum, etc...)
- Obviously you don't pay the price if you don't want to. (Features added/removed at linking time)
- Metadata can also concern compile options, target platform, version of compiler, version of library, etc...
- Reflection is not only about enumerating things but calling/reading/writing things.
  
## Idea 3 : Remove some mistakes/dangers from original C.
- Impossible to do assignment in conditionnal parameter : how many times people have been bitten by = instead of == inside a if / while / for loop.
  - Yes, you will write more code.
  - Yes, it will be less compact.
  - No, performance won't suffer : it is a compiler's job to fix that.
- Switch 'case' going through others are possible BUT EXPLICIT : through; <--- 'through' keyword is added to the language.
  - Again, avoid being bitten by forgetting break;
- Function and procedure are two different things (procedure = function returning void)
  - Ignoring the result of a function call returning a value will require explicitely 'ignore' keyword.
- bool are not int. 'if (int)' becomes 'if (int != 0)' like in Pascal or C#.
  - Therefore if/for/while condition are boolean.
  - Compiler responsability to get same perf as C.
- Cast are all explicit else treated as error (signed<->unsigned) or (bigger->smaller) or (int<->float). Again you type more. Yes it is annoying when you are used to C, but leaves no implicit issue unverified by the programmer. (Same as C#)
  
## Idea 5 : Visibility
- public   : visible from everywhere.
- internal : public from within the namespace, private outside.
- private  : visible from the struct itself only.

## Axis 6 : Boxed types
- All primitives types support object like feature. (float.Biggest, float.Smallest, float.Parse(...), like C#)
  - Same memory size, no difference from compiler perspective.
  - Nicer API.
- String would fit nicely. (.toLength(), toCStr(), ...)
  
## Axis 7 : Ability to extend class outside of the package.
- extends struct A { void myFunction(ref A this, ...my parameters...) { code } }
  - Useful to extend original things like string or float package. Ex : float.parse or string.ToUtf8() etc...
  - Unable to access internals from the package and only the object as parameter ? (First version ?)
  
## Axis 8 : Code Generation
- .obj code generation ==> LLVM I think. Delegate the whole shenanigans to a well established backend.

## Axis 9 : Structures
- Optionnal 'offset' tag : decide position in byte offset inside structure.
- Optionnal 'order' tag : does not decide position exactly but sorted by order. (allow source order and memory order to be different)
- Optionnal structure align tag : define if a struct need alignement.
- Optionnal 'align' tag inside struct for a member : require struct to be aligned too. And then perform padding relative to struct.
- What about union ? I think those are more dangerous than anything, yet useful. By allowing offset, we can also allow such behaviour.
  - 'override offset' should make it explicit when two members share the same memory.
  - No support in first draft and then can add back the C union later if needed ?

## On the topics of strings.
- Not a fat pointer. (Length + Ptr as a struct)
- Want to store length AND store a ZERO at the end (but length return without the zero).
  - Can just get raw ptr and use it as standard C string.
- strlen becomes O(1).
- Structure : [Size U32][Array of byte...]
- UTF8 the default.
- Design problem : The string pointer point to the array of byte (and we always do ptr-4 to get size) or point to size and we always do (ptr+4) to get access to the string.
- Is 4GB string too small ?
- Memory management of strings... Want to avoid the mess of C/C++ with char*.
  - Pass allocator when creating the first string. Then stuff like a_string + b_int use same allocator/free.
  - But what happen with a_string + b_string ?
  - Operator + issue. => Don't want to have overloading operators to support. (Special concatenation operator ? Ex : a + b + 3 => a$b$3 or a#b#3)

## Function pointers.
- Need those for DLL and C communication. Provide and use C function pointer. Allows also to create complex mecanism (COM object), or even C++ objects.

## Lambda Functions.
- No idea yet.

## Difference between pointer and array.
- Use generic array class and span.
- Pointer should only point to a single thing.
- Pointer arithmetic still authorized.

## Coalescing malloc/free.
- Borrowed idea from Jay language.
- It is not rare to allocate a bunch of various stuff with a single malloc to reduce memory allocation overhead in C.
- Error prone because manually computing things.
- Have the malloc/free setup a whole bunch of members instead of single member.

## Personal preferences.
- I think C syntax is well known, and there are no real benefit in doing a completely different syntax (like Jay or Rust are doing).
  - Ex : changing 'int myVar' to 'myVar int' (Parser efficiency ?)
  - Fn, let, etc... why ? Easier compiler ?
- 'auto' like feature will not exist. If typing is too long, define a shortcut using typedef. Everything explicit.
   - ex : thisPackage::struct::internalStruct  ==> typedef thisPackage::struct::internalStruct iStruct and voila !
   - Of course, if no name conflict, short name is OK (ex #import pa::pb::pc with struct d inside can directly call d)
- Add a 'nodefault' to 'default' possible switch cases : you garantee that the input you provide to the switch case is VALID for all cases (performance) and invalid values are UB (undefined behavior). (You own the responsability in exchange of performance gain explicit in the language. No more 'unreachable' compiler special magic)
- sizeof available at compile time for conditionnal macrolike setup.
- Enum can be read as string too.  EnumA value; --> value.ToString()
- No exceptions.
- Memory allocation customization. Want the allocator to be replacable everywhere. Able to plug your own allocator in already existing libraries.
- Memory alignment is fist citizen too. Defined in the language. (member, struct)
- SIMD a first citizen (intrinsic support or inline assembly if you don't trust register allocators and optimizer from intrinsics)
- Constant can use _ keyword to delimit and increase readability. ex : 0xDEAD_BE_EF
  - Binary constant are supported. ex : 0b0010_1100 (printf like formatting too)
- Read bit block inside a word like var[14:23] instead of making manually and masks. 
- No operator overloading.
  - Yep, it makes it harder for Vector3 / Vector4 / Complex numbers.
    - Or explicit  a + b + c => a .+ b .+ c ? Equivalent to a.add(b).add(c) kind of... ? 
  - But NO hidden complexity for the programmer. Reader must see that something is happening under the hood.
- #incbinary type endianness <file> into array definition. (#include for text file is still possible but #include inside #include is not)
  - Allow to build binary table and include those without having to generate those doing printf.
  - Pass struct type and original endianness. Importer manages to modify byte order.(Except of struct with union like definitions : forbidden)
  
## Undefined : OOP
- VTable / Trait like stuff...
  - I don't like the idea of 'fat' pointer. TWO pointer maintenance cost, even only if virtual is...
- AoS vs SoA.
  - Jay language is the one doing interesting things.
  - Would be nice to be able to define 'walking operator' : why not also morton code, etc... 

## Undefined or have no idea yet
- Are macro a good thing ? C# seems too limiting. But would like to have the possibility for the compiler to execute the AST of any package and interpret this language. Allowing full C/C++ macro like stuff and manipulate the raw text seems 'too much'.
  - Have the ability for macro to define a block of code like a template is nice.
  - Have the ability to remove or select piece of code or macro is nice.
  - Replacing name with other name / modify raw text is evil (ex : #define true false )
  => #define #if etc... should be considered like a statement from the source language.
- Inline assembler / ability to use assembler is a must. But what is the most convenient ? How to handle multiplatform as nicely as possible.
- Pointers, span and arrays.
  - Pointing to something and an array should be two different concepts. In one case you have only ONE item (like a reference), in the other case you delimit ranges and do some arithmetic,indexing on it.
- C has just pointers. C++ has reference added.
  - Reference is basically pointers again, except that NULL can not occur.
  - If reference supported, C# approach is very clean : in/out/ref usage is fully described BOTH at caller and callee. Forces initialization.
- Templates ? Generics ?
  - It is nice to be able to write an algorithm with passing the type and avoid doing copy paste of code. I'd like to keep that.
  - Things like array, hashmap, etc.. Need some generic way of handling those.
  - But I want to avoid to make impossible template metaprogramming.
- memory fence/ordering/volatile
- multithreading / fibers
- GPU ? Future of heterogeneous computing ? Write code for different system within ? ==> Metatags can come handy.
- Bit support inside struct like C does ?

## Standard libraries
- printf formatting is adapted to {1} number based, allowing to read the parameters orders the way you want. Other formatting parameters seems nice to keep.
  - Do we keep full compatibility for printf like stuff (porting all code)
