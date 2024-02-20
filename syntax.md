# Syntax

```rust
_expr-or-braced-init-list_ match requires(opt) constexpr(opt) _trailing-return-type_(opt)  {
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
    if _expression_

_pattern_:
    _match-pattern_
    let _binding-pattern_
    _match-pattern_ let _binding-pattern_

_match-pattern_:
    _                              // wildcard
    _constant-expression_          // equality match
    ( _pattern_ )                  // grouping
    ? _pattern_                    // if (e) then match *e
    _type-id_ : _pattern_          // dynamic types, e.g. std::variant / polymorphic types
    _type-constraint_ : _pattern_  // constraint match
    [ _sb-pattern-0_, /* ... */, _sb-pattern-N_ ]  // e.g., [let x, 0]
    [ _designator-0_ : _pattern-0_, /* ... */, _designator-N_ : _pattern-N_ ]  // e.g., [.foo: let x, .bar: 0]
    or ( _pattern-0_, /* ... */, _pattern-N_ )  // match one of. short-circuting.

_sb-pattern_:
    _pattern_
    ...

_binding-pattern_:
    _identifier_
    [ _sb-binding-pattern-0_, /* ... */, _sb-binding-pattern-N_ ]
    [ _designator-0_ : _binding-pattern-0_ , /* ... */, _designator-N_ : _binding-pattern-N_ ]

_sb-binding-pattern_:
    _binding-pattern_
    ... _identifier_(opt)
```