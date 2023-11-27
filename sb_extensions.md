# Structured Bindings Extensions

The following is the syntax for _binding-pattern_:

```rust
_binding-pattern_:
    _
    _identifier_
    ... _identifier_(opt) // only in positional structured bindings
    [ _binding-pattern0_ , _binding-pattern1_ , /* ... */, _binding-patternN_ ]
    [ _designator0_ : _binding-pattern0_ , /* ... */, _designatorN_ : _binding-patternN_ ]
```

The idea is to use this syntax consistenly between a variable declarations
and `let` bindings in `match` expressions. This should "retrofit" into
existing syntax seamlessly as well.

```rust
auto _binding-pattern_ = expr;

_pattern_:
    let _binding-pattern_
    // ...
```

# Examples

```cpp
// existing syntax
auto x = e;
auto [x, y] = e;
auto [_, y] = e;
auto [...xs] = e;
auto [...xs, last] = e;
// new syntax
for (auto const& [k, [v1, v2]] : dict) {
  // ...
}
auto [.quot: q, .rem: r] = div(-5, 3);
auto [x, [.foo: foo, .bar: bar], z] = e;
```

> The `div()` function computes the value numerator/denominator and returns
> the quotient and remainder in a structure named `div_t` that contains two
> integer members __(in unspecified order)__ named `quot` and `rem`.
>
> Source: https://linux.die.net/man/3/div
> Credit: Barry Revzin

The same syntax is used for after `let` in a `match` expression.

```rust
e match {
    let x => // ...
    let [x, y] => // ...
    let [_, y] => // ...
    let [...xs] => // ...
    let [...xs, last] => // ...
    let [k, [v1, v2]] => // ...
    let [.quot: q, .rem: r] => // ...
    let [x, [.foo: foo, .bar: bar], z] => // ...
}
```

## Why `[.quot: let q]`, and not `[let q = .quot]` or something like that?

`[let q = .quot]` actually does look reasonable, but it looks reasonable
only for the `let` case. For something like `[.quot: 42]`, we'd end up
with `[42 = .quot]` which doesn't look so good.

## Why not support accessors, e.g., `[.foo(): pattern1, .bar(): pattern2]`?

Accessors like this are different than public members in that they often have
preconditions. This makes allowing such code at a higher risk for introducing
bugs. For example, something like `[.value(): let v]` is risky since the object
may be in an unengaged state.