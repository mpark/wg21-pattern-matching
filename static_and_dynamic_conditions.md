# Static and Dynamic Conditions

Every pattern has a set of static and dynamic conditions.
For the most part, static conditions are simply `requires` of the dynamic conditions.
By default, the static conditions are checked (think `static_assert`) and dynamic
conditions are tested at runtime (think `if`).

`requires` and/or `constexpr` can be added to `match` to control the handling
of static and dynamic conditions.

Let's walk through a simple example:

```rust
void f(int x) {
  x match {
    0 => // ...
    1 => // ...
    _ => // ...
  };
}
```

Here, we have an expression pattern `0` whose static condition is
`requires { _subject_ == 0 }`, and the dynamic condition is `_subject_ == 0`.

Here, the semantics of this is basically:

```cpp
void f(int x) {
  if (x == 0) // ...
  else if (x == 1) // ...
  else // ...
}
```

This doesn't change even if we're matching against a `constexpr` value. For example,

```rust
template <int x>
void f() {
  x match {
    0 => // ...
    1 => // ...
    _ => // ...
  };
}
```

This still just has `if` semantics:

```cpp
template <int x>
void f() {
  if (x == 0) // ...
  else if (x == 1) // ...
  else // ...
}
```

But perhaps we want to test the dynamic conditions at compile-time.
`match constexpr` achieves this by making the dynamic conditions
be tested with `if constexpr`.

```rust
template <int x>
void f() {
  x match constexpr {
    0 => // ...
    1 => // ...
    _ => // ...
  };
}
```

This means:

```cpp
template <int x>
void f() {
  if constexpr (x == 0) // ...
  else if constexpr (x == 1) // ...
  else // ...
}
```

What if the `x` were a templated parameter instead, like this?

```rust
void f(auto x) {
  x match {
    0 => // ...
    1 => // ...
    _ => // ...
  };
}
```

Same as before, this still just:

```cpp
void f(auto x) {
  if (x == 0) // ...
  else if (x == 1) // ...
  else // ...
}
```

Specifically, this means that `f("hello"s)` upon instantiation is ill-formed.
But perhaps you want to be able to handle the `string` case as well. `f` is
a function template after all.

`match requires` is used in a situation like this to turn the static conditions
from being checked to being tested.

```rust
void f(auto x) {
  x match requires {
    0       => // A
    "hello" => // B
    _       => // C
  };
}
```

Concretely, this would be similar to:

```cpp
void f(auto x) {
  if constexpr (requires { x == 0; }) {
    if (x == 0) {
      // ...
      goto done;
    }
  }
  if constexpr (requires { x == "hello"; }) {
    if (x == "hello") {
      // ...
      goto done;
    }
  }
  // ...
  done:;
}
```

In the most advanced usage, these can be combined into `match requires constexpr` such that
the static conditions are tested instead of checked, and dynamic conditions are tested at compile-time.

|                     | without `requires`    | with `requires`                                           |
| ------------------- | --------------------- | --------------------------------------------------------- |
| without `constexpr` | if (_cond_)           | if constexpr (requires { _cond_; }) if (_cond_)           |
|    with `constexpr` | if constexpr (_cond_) | if constexpr (requires { _cond_; }) if constexpr (_cond_) |

# Examples

```rust
void f(int x) {
    x match {
        0 => do { std::print("got zero"); };
        1 => do { std::print("got one"); };
        _ => do { std::print("don't care"); };
    };
}
```

```rust
struct S {
    T x;
    U y;

    template <std::size_t I, typename Self>
    auto&& get(this Self&& self) {
        return I match constexpr -> auto&& {
            0 => std::forward<Self>(self).x;
            1 => std::forward<Self>(self).y;
            _ => throw std::out_of_range("");
        };
    }
}
```

```rust
template <typename T>
void f(T p) {
    p match requires {
        let [x, y] => do { std::print("2D point at {}, {}", x, y); };
        let [x, y, z] => do { std::print("3D point at {}, {}, {}", x, y, z); };
        _ => do { std::print("unexpected"); };
    };
}
```

```rust
template <auto X>
void f() {
    constexpr bool is_2d_origin = X match requires constexpr {
        [0, 0] => true;
        _ => false;
    };
}
```