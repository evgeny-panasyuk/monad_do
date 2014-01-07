monad_do
========

Emulation of `DO` syntax sugar for monads in C++

Code example
============

[**Live Demo**](http://coliru.stacked-crooked.com/a/2478cec9e7fcd62d)
```C++
#include <boost/optional.hpp>

#include <iostream>
#include <utility>
#include <cassert>

using namespace std;
using namespace boost;

// Identity Monad:
template<typename T>
struct Identity
{
    T data;
    template<typename F> friend auto operator>>=(Identity m, F f)
    {
        return f(m.data);
    }
};

// Maybe Monad:
template<typename F, typename T>
auto operator>>=(optional<T> m, F f)
{
    return m ? f(m.get()) : m;
}

template<template<typename> class Monad>
auto run()
{
    return
        DO(Monad,
            (x, unit(1))
            (y, unit(2))
            (z, DO(Monad,
                (x, unit(5))
                (_, unit(x - 2))
            ))
            (_, unit(x + y + z))
        );
}

int main()
{
    assert( run<Identity>().data == 6 );
    assert( run<optional>().get() == 6 );
    assert
    (
        not DO(optional,
            (x, unit(1))
            (y, unit(2))
            (z, DO(optional,
                (x, make_optional(false, 5))
                (_, unit(x - 2))
            ))
            (_, unit(x + y + z))
        )
    );
    cout << "DONE" << endl;
}
```