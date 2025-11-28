# Lynn's C++ notes
_Work in progress — might `#include <mistakes>`._

## Contents

1. [**Value Categories**](#value-categories--all-categories-have-value)
2. [**Move Semantics**](#move-semantics--now-for-my-next-move)

## Essentials

### [Value Categories](https://en.cppreference.com/w/cpp/language/value_category.html) / <sub>_All categories have value._</sub>
|                          | glvalue <br>(identity) | <br>(identityless) |
|--------------------------|------------------------|--------------------|
| **rvalue <br>(movable)** | `xvalue`               | `prvalue`          |
| **(immovable)**          | `lvalue`               |                    |

`glvalue`: (generalised lvalue) values with an _identity_, encapsulating `lvalues` and `xvalues`.
<br>`rvalue`: (read/right value) values that can be moved, encapsulating `xvalues` and `prvalues`.
<br>`lvalue`: (locator/left value) values that cannot be moved.
<br>`xvalue`: (eXpiring value) the result of applying `std::move` to an `lvalue`.
<br>`prvalue`: (pure rvalue) values without an _identity_.

- Here I have provided a list of most instances of each value category:

```cpp
// lvalues
int i;          // Named variables are lvalues
"Hello World!"; // String literals have lvalue type 'const char(&)[N]'
*this;          // The dereferenced 'this' pointer is an lvalue
void fn() {}    // 'fn' is an lvalue
struct { static void fn() {} }; // Static member functions are lvalues

// xvalues
std::move( i );

// prvalues
42; true; nullptr; // Literals are prvalues (except for string literals)
enum { val };      // Enum values are prvalues
i++;               // Post-increment/decrement expressions are prvalues
&i;                // A variable's address is a prvalue
int{};             // Temporary materialisation results in a prvalue
this;              // The 'this' pointer is a prvalue
struct { void fn() {} }; // Non-static member functions are prvalues
// Arithmetic, logical, and comparison expressions also result in prvalues
// Note that the above refers to built-in operations, not user-defined overloads
```

- Conversions always result in a `prvalue`, demonstrated with the following code snippet:

```cpp
void fn( int&& ) {}
void fn( float&& ) {}

int main()
{
    int i{};
    // 'i' cannot find an appropriate 'fn' overload taking an lvalue 'int'
    // 'i' can be implicitly converted to 'float&&'
    // Thus, 'fn( float&& )` is invoked
    fn( i );
}
```

> Values have an **_identity_** if they have an accessible address.
> <br>This includes variables, the dereferenced `this` pointer, string literals, etc.
> <br>Values without identity (`prvalues`) include literals, the `this` pointer, temporary objects (often as the result of an expression), etc.
> <br>Such values seize to exist if they are not defined.

> **_Moving_** a variable refers to reusing its resources to construct/assign another object.
> <br>For example, ownership of data pointers is transferred, instead of making a copy of the data.
> <br>Continue reading about move semantics [here](#move-semantics--now-for-my-next-move).

### [Move Semantics](https://en.cppreference.com/w/cpp/utility/move.html) / <sub>_Now for my next move..._</sub>

`std::move`: converts `lvalues` to `xvalues`. 
<br><sub>[std::move](https://en.cppreference.com/w/cpp/utility/move.html) [|]() [reference collapsing](https://en.cppreference.com/w/cpp/language/reference.html)</sub>

- `std::move` turns an `lvalue` into an `rvalue reference` by removing the current reference (if any) and adding `&&`.
    - The current reference must be removed in case the value is an `lvalue reference`, because _reference collapsing_ rules tell us `& &&` becomes `&`, while we want an `rvalue reference` (the symbol for which is `&&`).
    - `std::move` is equivalent to `static_cast<remove_reference_t<T>&&>`.
- A copy constructor/assignment operator can take in an `rvalue` if no move constructor/assignment operator is defined. Some STL classes will only use moves if they are marked `noexcept`, and will otherwise prefer the (non-throwing) copy operation.

```cpp
struct A
{
    A() : size{ 100 }, data{ new char[size] } {}
    ~A() { delete[] data; }

    // Copy constructor
    // Takes in an lvalue
    A( const A& other )
    {
        size = other.size;
        if (other.data)
        {
            data = new char[size];
            std::memcpy( data, other.data, size );
        }
        else
        {
            data = nullptr;
        }
    }
    // Move constructor
    // Takes in an rvalue (xvalue and prvalue)
    A( A&& other )
    {
        size = other.size;
        data = other.data;
        other.data = nullptr;
        other.size = 0;
    }

    std::size_t size{ 0 };
    char* data{ nullptr };
};

int main()
{
    A temp{} // Assume we have an object we don't need anymore
    A a1{ temp }; // 'temp' is an lvalue
    A a2{ std::move( temp ) }; // 'std::move( temp )' is an rvalue
}
``` 

`std::forward`: preserves the value category of a `forwarding reference`.
<br>`Forwarding references`: a parameter preserving value category in tandem with `std::forward`.
<br><sub>[std::forward](https://en.cppreference.com/w/cpp/utility/forward.html) [|]() [forwarding references](https://en.cppreference.com/w/cpp/language/reference.html) [|]() [reference collapsing](https://en.cppreference.com/w/cpp/language/reference.html)</sub>

- `Forwarding references` are also known as `universal references`.
- Both are used for templated functions, when you want to take in any value category.
- `Forwarding references` turn into an `lvalue` when referenced/used, except if they are wrapped in a `std::forward`.
- `std::forward` is equivalent to `static_cast<T&&>`, because of _reference collapsing_ rules.

```cpp
template <typename T>
void fn( T&& forwarding_ref )
{
    // 'forwarding_ref' keeps the value category passed to 'fn'
    other_fn( std::forward<T>( forwarding_ref ) );
    
    // 'forwarding_ref' is passed as an lvalue
    other_fn( forwarding_ref );
}

// Alternatively (since C++20)
void fn( auto&& forwarding_ref )
{
    using T = decltype( forwarding_ref );
    other_fn( std::forward<T>( forwarding_ref ) );
}
```

> **_Reference collapsing_** rules tell us the resulting reference type when making a reference to a reference (either through templates or `typedef`s). Only `rvalue reference` to `rvalue reference` collapses to an `rvalue reference`. Other combinations result in an `lvalue reference`.
>
> ```cpp
> using lref = int&;
> using rref = int&&;
>
> int i{};
> lref&  r1{ i }; // &  &  -> &
> lref&& r2{ i }; // &  && -> &
> rref&  r3{ i }; // && &  -> &
> rref&& r4{ i }; // && && -> &&
> ```

### Function (Pointers)
_At this point I'm barely functioning..._

### Inheritance
_Base(d).

### Casts
_You C-ing my style?_

`C-style Casts`: removed in C++23 (this is a joke, but please stop using it).

</details>

## bool operator<( Essential )
_Less than essential._

### Exception handling

### Decltype

### New & Delete

### Templates

## C++ Standard Library

## C++(++)
 _(beyond C++)_

### Compilation

### Computer Architecture
