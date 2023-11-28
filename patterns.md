# Patterns

```rust
_match-pattern_:
    _                              // wildcard
    _constant-expression_          // equality match
    ( _pattern_ )                  // grouping
    ? _pattern_                    // if (e) then match *e
    < _discriminator_ > _pattern_  // dynamic types, e.g. std::variant / polymorphic types
    [ _sb-pattern0_, /* ... */, _sb-patternN_ ]  // e.g., [let x, 0]
    [ _designator0_ : _pattern0_ , /* ... */, _designatorN_ : _patternN_ ]  // e.g., [.foo: let x, .bar: 0]
    or ( _pattern0_, /* ... */, _patternN_ )  // match one of. short-circuting.
_sb-pattern_:
    _pattern_
    ...
```

## Wildcard Pattern

> `_`

- Static Condition: None
- Dynamic Condition: None

## Expression Pattern

> _constant-expression_

- Static Condition: `std::equality_comparable_with<decltype(_subject_), decltype(_constant-expression_)>`
- Dynamic Condition: `_subject_ == _constant-expression_`

## Optional Pattern

> `?`

- Static Condition: `_boolean-testable_<decltype(_subject_)>`
- Dynamic Condition: _subject_ converts to true

> `?` _pattern_

- Static Condition: `_boolean-testable_<decltype(_subject_)> && requires { *_subject_ }`
- Dynamic Condition: _subject_ converts to true and `*`_subject_ matches _pattern_

## Structured Bindings Pattern

> `[` _sb-pattern0_`, /* ... */,` _sb-patternN_ `]`

```rust
_sb-pattern_:
    _pattern_
    ...
```

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
```

```cpp
structured_bindable<
    std::remove_reference_t<decltype((_subject_))>,
    /* has an ellipsis (...) */,
    /* number of identifiers */>();
```

- Dynamic Condition:

_eI_ matches _sb-patternI_ for all _I_ where _eI_ is an identifier,
given the following structured binding declaration:

`auto&& [` _e0_`, /* ... */,` _eN_ `] = ` _subject_;

where _eI_ is a unique exposition-only identifier if _sb-patternI_ is a _pattern_
and an ellipsis (`...`) if _sb-patternI_ is an ellipsis (`...`).

> `[` _designator0_ `:` _pattern0_`, /* ... */,` _designatorN_ `:` _patternN_ `]`

- Static Condition:

```cpp
using E = std::remove_reference_t<decltype((_subject_))>;
```

`std::is_class_v<E>` is true and `E` has non-static accessible members referred to by _designator_.

- Dynamic Condition: _subject_ _designatorI_ (e.g., `x.foo`) matches _patternI_ for all _I_
