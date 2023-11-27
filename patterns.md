# Patterns

```rust
_match-pattern_:
    _                              // wild card
    ...                            // only in positional structured bindings
    _constant-expression_          // equality match
    ( _pattern_ )                  // grouping
    ? _pattern_                    // if (e) then match *e
    < _discriminator_ > _pattern_  // dynamic types, e.g. std::variant / polymorphic types
    [ _pattern0_, /* ... */, _patternN_ ]  // e.g., [let x, 0]
    [ _designator0_ : _pattern0_ , /* ... */, _designatorN_ : _patternN_ ]  // e.g., [.foo: let x, .bar: 0]
    or ( _pattern0_, /* ... */, _patternN_ )  // match one of. short-circuting.
```

## Wildcard Pattern

> _

Static Condition: None
Dynamic Condition: None

## Expression Pattern

> _constant-expression_

Static Condition:

```cpp
std::equality_comparable_with<
    decltype(_subject_),
    decltype(_constant-expression_)>
```

Dynamic Condition: `_subject_ == _constant-expression_`

## Optional Pattern

> ? _pattern_

Static Condition:

```cpp
requires {
  requires _boolean-testable_<decltype(_subject_)>;
  { *_subject_ };
}
```

Dynamic Condition:

If _subject_ converts to true, then match _pattern_ against `*`_subject_