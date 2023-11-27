# Syntax

```rust
_expr-or-braced-init-list_ match constexpr(opt) _trailing-return-type_(opt)  {
    _pattern_ _guard_(opt) => _expr-or-braced-init-list_ ;
    _pattern_ _guard_(opt) => break ;
    _pattern_ _guard_(opt) => continue ;
    _pattern_ _guard_(opt) => return _expr-or-braced-init-list_ ;
    // NOTE: `goto` is excluded.
}

_expr-or-braced-init-list_ match constexpr(opt) _pattern_ _guard_(opt)
```

```rust
_guard_:
    if constexpr _constant-expression_ // static guard
    if _expression_                    // dynamic guard

_pattern_:
    _match-pattern_
    let _binding-pattern_
    _match-pattern_ let _binding-pattern_

_match-pattern_:
    _                              // wildcard
    ...                            // only in positional structured bindings
    _constant-expression_          // equality match
    ( _pattern_ )                  // grouping
    ? _pattern_                    // if (e) then match *e
    < _discriminator_ > _pattern_  // dynamic types, e.g. std::variant / polymorphic types
    [ _pattern0_, /* ... */, _patternN_ ]  // e.g., [let x, 0]
    [ _designator0_ : _pattern0_ , /* ... */, _designatorN_ : _patternN_ ]  // e.g., [.foo: let x, .bar: 0]
    or ( _pattern0_, /* ... */, _patternN_ )  // match one of. short-circuting.

_binding-pattern_:
    _identifier_
    ... _identifier_(opt) // only in positional structured bindings
    [ _binding-pattern0_ , _binding-pattern1_ , /* ... */, _binding-patternN_ ]
    [ _designator0_ : _binding-pattern0_ , /* ... */, _designatorN_ : _binding-patternN_ ]

_discriminator_: one of
    auto, type-constraint, type-id, constant-expression
```