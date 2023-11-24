Suppose we go with `match` always being an expression, like this:

_expression_ `match` `{`
    _pattern1_ `=>` _expression1_ `;`
    _pattern2_ `=>` _expression2_ `;`
    `// ...`
`}`

Let's say we have statement-expressions available in the form of
`do` expression our disposal.

```rust
auto x = do -> std::optional<int> {
    LOG(INFO) << "Some statements";
    do_return 42;
};
```

The following are a few scenarios:

1.1 Yield values of the matching type.

This is the nice case.

```rust
int x = v match {
    <int> let i => i;
    <std::string> let s => std::stoi(s);
};
```

1.2 Return values of the matching type.

```rust
int f() {
    // ...
    return v match {
        <int> let i => i;
        <std::string> let s => std::stoi(s);
    };
}
```

2.1 Yield values of different types.

This is still okay.

```rust
auto x = v match -> std::optional<int> {
    <int> _ => std::nullopt;
    <std::string> let s => std::stoi(s);
};
```

2.2 Return values of different types.

```rust
std::optional<int> f() {
    // ...
    return v match -> std::optional<int> {
        <int> _ => std::nullopt;
        <std::string> let s => std::stoi(s);
    };
}
```

3.1 Yield `do` expressions of the matching type.

```rust
int x = v match {
    <int> let i => do {
        LOG(INFO) << "int";
        do_return i;
    };
    <std::string> let s => do {
        LOG(INFO) << "str";
        do_return std::stoi(s);
    };
};
```

3.2. Return `do` expressions of the matching type.

```rust
int f() {
    // ...
    return v match {
        <int> let i => do {
            LOG(INFO) << "int";
            do_return i;
        };
        <std::string> let s => do {
            LOG(INFO) << "str";
            do_return std::stoi(s);
        };
    };
}
```

4.1 Yield `do` expressions of different types.

```rust
auto x = v match -> std::optional<int> {
    <int> _ => do {
        LOG(INFO) << "int";
        do_return std::nullopt;
    };
    <std::string> let s => do {
        LOG(INFO) << "str";
        do_return std::stoi(s);
    };
};
```

4.2. Return `do` expressions of different types.

```rust
std::optional<int> f() {
    // ...
    return v match -> std::optional<int> {
        <int> _ => do {
            LOG(INFO) << "int";
            do_return std::nullopt;
        };
        <std::string> let s => do {
            LOG(INFO) << "str";
            do_return std::stoi(s);
        };
    };
}
```

This relies on each `do`-type being convertible to the `match`-type.
It starts to suck when the return isn't value-based, convertible like this.

The trailing return type needs to be specified at all 3 levels:
return type, `match` type, and the `do` type.

Even with just `auto&`, it gets annoying:

```rust
std::ostream& f(std::ostream& strm) {
    // ...
    return v match -> auto& {
        <int> let i => do -> auto& {
            LOG(INFO) << "int";
            do_return strm << i;
        };
        <std::string> let s => do -> auto& {
            LOG(INFO) << "str";
            do_return strm << s;
        };
    };
}
```

It ends up being that the "statement version" with just `match` / `do` is better.

```rust
std::ostream& f(std::ostream& strm) {
    // ...
    v match {
        <int> let i => do {
            LOG(INFO) << "int";
            return strm << i;
        };
        <std::string> let s => do {
            LOG(INFO) << "str";
            return strm << s;
        };
    };
}
```