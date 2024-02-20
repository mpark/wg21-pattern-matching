# Patterns

```rust
_match-pattern_:
    _                            // wildcard
    _constant-expression_        // equality match
    ( _pattern_ )                // grouping
    ? _pattern_                  // if (e) then match *e
    _discriminator_ : _pattern_  // dynamic types, e.g. std::variant / polymorphic types
    [ _sb-pattern-0_, /* ... */, _sb-pattern-N_ ]  // e.g., [let x, 0]
    [ _designator-0_ : _pattern-0_ , /* ... */, _designator-N_ : _pattern-N_ ]  // e.g., [.foo: let x, .bar: 0]
    or ( _pattern-0_, /* ... */, _pattern-N_ )  // match one of. short-circuting.

_sb-pattern_:
    _pattern_
    ...
```

## Wildcard Pattern

> `_`

- Static Condition: None
- Dynamic Condition: None

## Constant Pattern

> _constant-expression_

- Static Condition: `requires { bool(_subject_ == _constant-expression_); }`
- Dynamic Condition: `_subject_ == _constant-expression_` contextually converts to true

## Parenthesized Pattern

> `(` _pattern_ `)`

- Static Condition: Static condition of: _subject_ matches _pattern_
- Dynamic Condition: Dynamic condition of: _subject_ matches _pattern_

## Optional Pattern

> `?` _pattern_

- Static Condition: _subject_ is contextually convertible to `bool` plus the static condition of: `*`_subject_ matches _pattern_
- Dynamic Condition: _subject_ is contextually converts to true and *_subject_ matches _pattern_

## Structured Bindings Pattern

> `[` _sb-pattern-0_`, /* ... */,` _sb-pattern-N_ `]`

```rust
_sb-pattern_:
    _pattern_
    ...
```

- Mandates: There must be at most one ellipsis (`...`) present.
- Static Condition: Same as structured bindings. Approximately:

```cpp
template <typename T, bool HasEllipsis, std::size_t NumIdentifiers>
constexpr bool structured_bindable() {
  constexpr auto compare = [](std::size_t size) {
    return HasEllipsis ? NumIdentifiers <= size : NumIdentifiers == size;
  };
  if constexpr (std::is_array_v<T>) {
    return compare(/* number of array elements */);
  } else if constexpr (std::is_class_v<T>) {
    if constexpr (requires { std::tuple_size<T>::value; }) {
      if constexpr (std::integral<decltype(std::tuple<T>::value)>) {
        return compare(std::tuple<T>::value);
      }
    } else {
      return compare(/* number of non-static data members */);
    }
  }
  return false;
}

structured_bindable<
    std::remove_reference_t<decltype((_subject_))>,
    /* has an ellipsis (...) */,
    /* number of identifiers */>()
```

- Dynamic Condition:

Given the following structured binding declaration:

`auto&& [` _e-0_`, /* ... */,` _e-N_ `] = ` _subject_;

where _e-i_ is a unique exposition-only identifier if _sb-pattern-i_ is a _pattern_
and an ellipsis (`...`) if _sb-pattern-i_ is an ellipsis (`...`),

_subject_ matches if _e-i_ matches _sb-pattern-i_ for all _i_ where _e-i_ is an identifier.

> `[` _designator-0_ `:` _pattern-0_`, /* ... */,` _designator-N_ `:` _pattern-N_ `]`

- Static Condition:

Let `E` be `std::remove_reference_t<decltype((_subject_))>`.

`std::is_class_v<E>` is true and `E` has non-static accessible members referred to by _designator_.

- Dynamic Condition: _subject_ _designator-i_ (e.g., `x.foo`) matches _pattern-i_ for all _i_

## Or Pattern

> `or (` _pattern-0_`, /* ... */,` _pattern-N_ `)`

- Mandates: Any bindings introduced must be present in every _pattern_
- Static Condition: Conjunction of the static conditions of: _subject_ matches _pattern-i_
- Dynamic Condition: _subject_ matches one of _pattern-i_, tested left-to-right