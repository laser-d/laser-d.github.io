Introduction
============

Laser-D is a computer programming language based on the D programming Language. The language differs from D in some major ways: Laser-D does not support 
garbage collection and does not provide a runtime. As a consequence, Laser-D does not support certain features of the D programming language. Laser-D 
does not support the standard D library phobos for the same reason.

Here is a hello world program in Laser-D.

::

    import core.stdc.stdio : printf;

    extern (C) void main()
    {
        printf("Hello World!\n");
    }


Basic Concepts
==============

Laser-D programs are structured as modules that can be compiled separately and linked with external libraries to create 
native libraries or executables.

Each Laser-D source file defines a single module. 

Each laser-D module contains one or more declarations of entities. 

An entity is a named variable, function, structure or union type, enumeration type, template, mixin template or an external symbol.

Entities in a Laser-D module can have linkage that specifies how these entities may be accessed from other Laser-D modules, or from programs in other 
other languages such as C or C++.

Source Code Representation
==========================

Source code must be Unicode text encoded in UTF-8.

Characters
----------

The following terms are used to denote specific Unicode character classes:

``newline`` 
    the Unicode code point U+000A

``unicode_char``
    an arbitrary Unicode code point except ``newline``

``universal-character-name``
    as defined in ISO/IEC 9899:1999(E) Appendix D of the C99 Standard

``nondigit``

    one of::
    
        a b c d e f g h i j k l m n o p q r s t u v w x y z
        A B C D E F G H I J K L M N O P Q R S T U V W X Y Z
        _

    universal-character-name

``digit``
    one of::

        0 1 2 3 4 5 6 7 8 9

The underscore character ``_`` (U+005F) is considered a letter.


Lexical Elements
================

Keywords
--------

The language has following keywords.

::

    abstract    alias        align          asm          assert      auto
    body        bool         break          byte         case        cast
    catch       cdouble      cent           cfloat       char        class
    const       continue     creal          dchar        debug       default
    delegate    delete       deprecated     do           double
    else        enum         export         extern       false       final
    finally     float        for            foreach      foreach_reverse    function
    goto        idouble      if             ifloat       immutable    import
    in          inout        int            interface    invariant    ireal
    is          lazy         long           macro        mixin        module
    new         nothrow      null           out          override     package
    pragma      private      protected      public       pure 
    real        ref          return         scope        shared       short
    static      struct       super          switch       synchronized
    template    this         throw          true         try          typeid
    typeof      ubyte        ucent          uint         ulong        union
    unittest    ushort       version        void         wchar        while
    with        __FILE__     __FILE_FULL_PATH__          __MODULE__   __LINE__
    __FUNCTION__             __PRETTY_FUNCTION__         __gshared    __traits     
    __vector    __parameters


Operators and Punctuation
-------------------------

::

    /     /=    .    ..    ...   &
    &=    &&    |    |=    ||    -
    -=    --    +    +=    ++    <
    <=    <<    <<=  >     >=    >>=
    >>>=  >>    >>>  !     !=    (
    )     [     ]    {     }     ?
    ,     ;     :    $     =     ==
    *     *=    %    %=    ^     ^=
    ^^    ^^=   ~    ~=    @    =>
    #


Identifiers
-----------

An identifier is a sequence chracters with a ``nondigit`` character followed by ``nondigit`` or ``digit`` characters.  

Lowercase and uppercase letters are distinct. There is no specific limit on the maximum length of an identifier.

Comments
--------

Comments serve as program documentation. There are two forms:

* Line comments start with the character sequence // and stop at the end of the line.
* General comments start with the character sequence /\* and stop with the first subsequent character sequence \*/.

String Literals
---------------

A string literal is either a double quoted string, a wysiwyg quoted string, a delimited string, a token string, or a hex string.

Names
=====
In a Laser-D program, identifiers can be used to denote:

* modules
* structures and union types
* template types and template parameters
* functions and function arguments
* aliases
* labels
* variables
* mixin templates
* import bindings
* attributes
* external symbols

The same identifier can denote different entities at different points in the program.

Scope
-----
For each different entity that an identifier designates, the identifier is visible (i.e., can be used) only within a region of program text 
called its scope. There are following kinds of scopes: global, module, struct or union type, template type, function, block. 

Different entities designated by the same identifier either have different scopes, or the identifiers must be in an overload set.


Modules
=======

Each Laser-D source file represents a distinct module. Each module has a name, optionally qualified with a package prefix.
The module name, if not declared explicitly using the ``module`` declaration, is derived from the source file name.  

If present, the ``module`` declaration must be the first and only such declaration in the source file, 
and may be preceded only by comments and #line directives.

::

    ModuleDeclaration:
        module ModuleFullyQualifiedName ;

    ModuleFullyQualifiedName:
        ModuleName
        Packages . ModuleName

    ModuleName:
        Identifier

    Packages:
        PackageName
        Packages . PackageName

    PackageName:
        Identifier

The fully qualified module name forms part of the qualified name of every entity declared in that module. Thus two entities with identical names in different
modules can be disambiguated using the fully qualified names of the entities.




