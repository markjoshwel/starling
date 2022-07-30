# This is Ultraviolet 0.1.0

> **Warning**
>
> Ultraviolet is still a proof of concept and should not be used in production, _ever_.

Ultraviolet is a novel programming language with roots from all over.

```uv
using std.print

print("Hello World!")
```

- [Installing](#installing-ultraviolet)
- [Using](#using-ultraviolet)
- [Developing](#developing-for-ultraviolet)
- [Learning](#learning-ultraviolet)
- [Licence](#licence)

## Installing Ultraviolet

**Requirements**

1. LLVM 11.x.x
2. Python 3.10.x

```text
pip install ultraviolet
```

## Using Ultraviolet

Run a Ultraviolet program:

```text
uv run program.uv
```

Type-check an Ultraviolet program:

```text
uv check program.uv
```

Format an Ultraviolet program:

```text
uv format program.uv
```

Formatting rules for the built-in formatter loosely follow the [Black code style](https://black.readthedocs.io/en/stable/the_black_code_style/current_style.html), with the main difference of 3 characters as the indentation unit.

## Developing for Ultraviolet

Ultraviolet source code, located in `src/ultraviolet` is seperated into multiple high-level files. Listed below are files of high importance, and the codebase can be further traversed from traversing module calls and imports.

- `src/ultraviolet/cli.py`

  Processing of command-line arguments.

- `src/ultraviolet/stage1.py`

  Parsing and processing of a Ultraviolet source file.

- `src/ultraviolet/stage2.py`

  Conversion of the processed source code in Stage 1 into LLVM IR where it is then compiled and executed just-in-time.

## Learning Ultraviolet

I assume you know the absolute basics of programming. If not, pick up some Python and come back in a week, or less.

Good news, if you are familar with Python's syntax, you're 10% done.

_Even better news_, if you are familar with static typing or [mypy](http://mypy-lang.org/), you're 10% more done.

- [Declarations](#declarations)
  - [Variable Declaration](#variable-declaration)
  - [Function Declaration](#function-declaration)

- [Constructs](#constructs)
  - [Selection Constructs](#selection)
  - [Iteration Constructs](#iteration)

- [Types and Typing](#types-and-typing)
  - [Primitive Types](#primitive-types)
    - [int](#int)
    - [char](#char)
    - [float](#float)
    - [bool](#bool)
    - [`none`](#none)
    - [tuple](#tuple)
    - [array](#array)
  - [Built-in Types](#built-in-types)
    - [`@List`](#list)
    - [`@String`](#string)
  - [String Literals](#string-literals)
  - [Annotating Collections](#annotating-collections)

- [Functions](#functions)
  - [Function Arguments](#function-arguments)
  - [Function Calling](#function-calling)

- [Semicolons](#semicolons)

- [Execution](#execution)

### Declarations

Declarations are the way to assign values to a variable or constant, and can also be used to define functions.

#### Variable Declaration

Format:

```text
<identifier>[: <type>] = <value>[;]
```

Examples:

```uv
message = "hello world";
message: str = "hello world"
```

More details about types are given in [Types and Typing](#types-and-typing).

#### Function Declaration

Format:

```text
<identifier>: func([<arg_name>: <arg_type>, ...]) -> <return_type> = <code>;
```

Note: If an argument is given, its type must also be provided. Return types may be inferred by the compiler.

Examples:

```uv
using std.print

main: func() =
   print("Hello World!")
;
```

```uv
add: func(x: int, y: int) -> int =
   return x + y
;
```

More details about types are given in [Types and Typing](#types-and-typing).
More details about function arguments are given in [Function Arguments](#function-arguments).

### Constructs

#### Selection

Format:

```text
if <expression>: <code> [elif <expression>: <code>] [else <expression>: <code>] ;
```

Examples:

```uv
using std.print

if timestamp == 1665849600:
   print("🍞")
;
```

```uv
using std.print

if number % 2 == 0:
   print("even")
elif number % 2 == 1:
   print("odd")
else:
   print("unreachable")
;
```

The seperation of code blocks occur when the `elif` or `else` keywords are reached by the parser. As such, semicolons are only required at the end of the if-elif-else statement.

#### Iteration

##### for-else Loop

The for-else can be used with generators.

Format:

```text
for <expression>: <code> [else: <code>] ;
```

Example:

```uv
using std.print

for char in @iter("hello there"):
   print(char)
;
```

The seperation of code blocks occur when the `else` keyword is reached by the parser. As such, semicolons are only required at the end of the for-else statement.

##### while-else Loop

Format:

```text
while <expression>: <code> [else: <code>] ;
```

Example:

```uv
using std

number = std.input("Enter a number: ")
while not std.ascii.isnumeric(number):
   std.print("Not a number!")
   number = std.input("Enter a number: ")
;
```

The seperation of code blocks occur when the `else` keyword is reached by the parser. As such, semicolons are only required at the end of the white-else statement.

### Types and Typing

#### Primitive Types

There are seven primitive types in Ultraviolet.

| Type            | Equivalent                      | Description                                                           |
|-----------------|---------------------------------|-----------------------------------------------------------------------|
| [int](#int)     | `int64_t` (C); `i64` (Zig, ...) | Signed 64-bit integer                                                 |
| [char](#char)   | `u8` (Zig)                      | Unsigned 8-bit integer mainly used for UTF-8 character representation |
| [float](#float) | `double` (C); `f64` (Zig, ...)  | IEEE-754-2008 64-bit floating point, 52-bit mantissa                  |
| [bool](#bool)   | `bool` (C, Zig, ...)            | Boolean value, `true` or `false`                                      |
| [`none`](#none) | `void` (Zig, ...)               | 0 bit type, used similarly as Python's `None`                         |
| [tuple](#tuple) | Rust Tuple                      | Fixed-size immutable collection of non-homogeneous type               |
| [array](#array) | Rust/Zig Array                  | Fixed-size collection of homogeneous type                             |

##### int

Integers (`int`) are signed 64-bit integers and have a range from `-9223372036854775808` to `9223372036854775807`.

```uv
min: int = -9223372036854775808
max: int =  9223372036854775807
```

##### char

Characters (`char`) are unsigned 8-bit integers and are used for character representation. Non-US-ASCII/Unicode code points require an array of `char`s.

```uv
ch1: char = "a"
ch2: array[char, char] = "糞"  # U+7CDE; requires two bytes.
ch3: array[char, ...7] = "🇸🇬"  # made up of regional indicators 'S' and 'G'; U+1F1F8 and U+1F1EC; made up of eight bytes.
```

##### float

Floating-point values (`float`) are stored as IEEE-754-2008 double-precision binary floating-points. (binary64) This gives them a range between approximately `1.7976931348623157e+308` and `4.9406564584124654e-324`.

```uv
funny: float = 69.42
```

##### bool

Booleans are a binary type, with the value of either `true` or `false`.

```uv
will_uv_ever_be_producion_ready: bool = false
```

##### none

`none` is a 0 bit type, used similarly as Python's `None`. It can be used to annotate a non-returning function, denote optional types or such.

```uv
main: func() -> none =  # unlike python, the function doesnt return 'none'
   ...
;
```

##### tuple

Tuples are fixed-size immutable collection of non-homogeneous type. Similar to Rust and Python, tuples are denoted by comma-seperated values surrounded by round brackets. (`()`)

```uv
cake_ingredient: tuple[int, str] = (1, "cross borehole electromagnetic imaging rhubarb")
```

For more information on denoting multiple instances of a type, see [Annotating Collections](#annotating-collections).

##### array

Arrays are fixed-size-at-compile-time collections of homogenous type. Similar to Rust, arrays are denoted by comma-seperated values surrounded by square brackets. (`[]`)

```uv
timestamps: array[int] = [1, 2, 3]
```

For more information on denoting multiple instances of a type, see [Annotating Collections](#annotating-collections).

#### Built-in Types

These types are necessarily loaded in by the compiler. As these types are part of the Ultraviolet `builtins` module, they are prefixed with an at sign. (`@`)

| Type      | Equivalent  | Description                                                 |
|-----------|-------------|-------------------------------------------------------------|
| `@List`   | Python List | Heap-allocated mutable collection of non-homogenous type    |
| `@String` | Rust String | Container of @List for `char`s with manipluation primitives |

##### @List

(Built-in Type) Lists are heap-allocated mutable collections of non-homogenous type

```uv
using std.print

a_list: @List = @List(1, "Two", 3.0)
print(a_list)  # [1, "Two", 3.0]
```

##### @String

(Built-in Type) Strings are a @List-based container for [`char`](#char) with manipulation primitives

#### String Literals

String literals in Ultraviolet are Unicode characters represented as an array of UTF-8 US-ASCII+Unicode code points, or in Ultraviolet typing terms, `array[char]`. These literals can be surrounded by single or double quotes.

Three single or double quotes will result in a multi-line string.

```uv
singleline: array[char...] = "This is a single line string!"
multiline: array[char...] = '''This is a
multi-line string!'''
```

#### Annotating Collections

Multiple instances of a type can be annotated using the elipsis keyword. (`...`)

1. `...<n>` after a colon denote that all elements leading up to the colon will repeat `n` more times.

   ```uv
   example1: tuple[array[char], int, ...2] = (1, "one", 2, "two", 3, "three")
   ```

2. `...<n>` after a type denote that this type will repeat `n` more times.

   ```uv
   # breakdown:    1----->  2----->     1----------------------->  2---------->
   example2: tuple[str...1, int...1] = ("First Name", "Last Name", 01, 01, 2000)
   ```

3. Square brackets can also be used to group types together. (This is **different** from union types!)

   ```uv
   # breakdown:    1--------->  2-------------------->  3->     1------->  2------------------------------------->  3
   example3: tuple[array[char], [array[char], int]...1, int] = ("current", "show1", 5.5, true, "show2", 7.5, false, 1)
   ```

### Functions

#### Function Arguments

TODO

#### Function Calling

TODO

### Semicolons

Semicolons, with the exception of [function declarations](#function-declaration), are not usually needed in Ultraviolet. However, they can be used after statements in order to minify lines or entire files.

Take the following program:

```uv
using std.print

main: func() =
   print("hello world")
;
```

Using semicolons, it can be minified to:

```uv
using std.print; main: func() = print("hello world");;
```

Note the double semicolon at the end. The first of the two semicolons is to end the `print("hello world")` statement. The second of the two semicolons is to end the function declaration.

### Execution

Ultraviolet programs have can have a `main()` function as a **command-line** entry point.

```uv
using std.print

print("this is ran everytime, be it module or command-line usage")

main: func() =
   print("this is only ran in command-line usage")
;
```

# Licence

The compiler and included tools (`src/ultraviolet/**/*`) are licenced under the Mozilla Public License 2.0. For more information, please refer to <https://www.mozilla.org/en-US/MPL/2.0/> or the [LICENCE](LICENCE) file in the repositories' root directory.

The standard library (`src/stdlib/**/*`) and `builtins` module (`src/builtins/**/*`) are free and unemcumbered software released into the public domain. For more information, please refer to <http://unlicense.org/> or the [UNLICENSE](UNLICENSE) file in the repositories' root directory.

In doubt, each Ultraviolet program file will have a comment header that details which licence it is under.
