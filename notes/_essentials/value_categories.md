---
title: "Value Categories"
layout: post
date: 1-1-1 # Using the first number as a sorting index
---

> _"All categories have value."_

#### Basics

|                          | glvalue <br>(identity) | <br>(identityless) |
|--------------------------|------------------------|--------------------|
| **rvalue <br>(movable)** | `xvalue`               | `prvalue`          |
| **(immovable)**          | `lvalue`               |                    |

- **glvalue:** (generalised lvalue) values with an _identity_, encapsulating `lvalues` and `xvalues`.
- **rvalue:** (read/right value) values that can be moved, encapsulating `xvalues` and `prvalues`.
- **lvalue:** (locator/left value) values that cannot be moved (only copied).
- **xvalue:** (eXpiring value) the result of applying `std::move` to an `lvalue`.
- **prvalue:** (pure rvalue) values without an _identity_.

Here I have provided a list of most instances of each value category:

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

Conversions always result in a `prvalue`, demonstrated with the following code snippet:

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