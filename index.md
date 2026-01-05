---
title: "Lynn's C++ Notes"
layout: home
permalink: /
cover:
---

# Testing

> Made this to avoid having to research the same concept over and over, you could say this is my cpp preference over cppreference :)

Work in progress — might `#include <mistakes>`.

# Oh ok ig

hiiii

$$ x^2 + y^2 = z^2 $$

```cpp
#include <iostream>

using namespace std;

std::vector<int> vec;

// This is a comment
class Obj
{
public:
    static constexpr int var{ 800 };
    static constexpr std::string s{ "Pom Ray Tracer" };
    int i;

    enum class MyEnum : std::uint8_t
    {
        ZERO = 0,
        ONE = 1
    };
};

void fn() {}

int main()
{
    if (true)
    {
        std::cout << sizeof( Obj::var ) << "Hello" << 'W';
        Obj obj{};
        obj.i = 3;
    }

    nullptr;
    NULL;

    a + b;
    c | d;
    x xor y;

    return 0;
}
```

# Theme notes:

- Figure out how to remove or set those links in the top right
- Find out what that A symbol is for

> tip
{: .block-tip }
> warning
{: .block-warning }
> danger
{: .block-danger }