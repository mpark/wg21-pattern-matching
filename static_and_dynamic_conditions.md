# Static and Dynamic Conditions

Every pattern has a set of static and dynamic conditions.
These conditions apply in different ways in different contexts.

Given the basic form of `match` expression:

```rust
expr match constexpr(opt) {
    pattern0 _guard_(opt) => expr0;
    pattern1 _guard_(opt) => expr1;
    // ...
}

_guard_
    if constexpr _constant-expression_
    if _expression_
```

If the _subject_ is non-dependent, static conditions of patterns are effectively
`static_assert(` _static-conditions_ `)`. If it is dependent, they have SFINAE
semantics, and are effectively `if constexpr (` _static-conditions_ `)`.

If it is a regular `match` expression, the dynamic conditions are matched at
runtime under `if (` _dynamic-conditions_ `)` semantics.
If we have a `match constexpr` expression, the dynamic conditions are matched
at compile-time under `if constexpr (` _dynamic-conditions_ `)` semantics.

|               | `match`                                   | `match constexpr`                                   |
| ------------- | ------------------------------------------| --------------------------------------------------- |
| non-dependent | `static_assert(_static_); if (_dynamic_)` | `static_assert(_static_); if constexpr (_dynamic_)` |
| dependent     | `if constexpr (_static_) if (_dynamic_)`  | `if constexpr (_static_) if constexpr (_dynamic_)`  |

The optional _guard_ also provides a trailing, free-form static and dynamic conditions.
They have the semantics of `if constexpr` or `if` depending on which one is used.

# Examples

```rust
void f(int x) {
    // non-dependent match
    x match {
        0 => do { std::print("got zero"); };
        1 => do { std::print("got one"); };
        _ => do { std::print("don't care"); };
    };
}
```

```rust
template <typename T>
void f(T x) {
    // dependent match
    x match {
        let [x, y] => do { std::print("2D point at {}, {}", x, y); };
        let [x, y, z] => do { std::print("3D point at {}, {}, {}", x, y, z); };
        _ => do { std::print("unexpected"); };
    };
}
```

```rust
struct S {
    T x;
    U y;

    template <std::size_t I, typename Self>
    auto&& get(this Self&& self) {
        // non-dependent match constexpr
        return I match constexpr -> auto&& {
            0 => std::forward<Self>(self).x;
            1 => std::forward<Self>(self).y;
            _ => throw std::out_of_range("");
        };
    }
}
```

```rust
template <auto X>
void f() {
    // dependent match constexpr
    constexpr bool is_2d_origin = X match constexpr {
        [0, 0] => true;
        _ => false;
    };
}
```