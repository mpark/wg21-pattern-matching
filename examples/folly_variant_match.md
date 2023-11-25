Example 1: https://github.com/facebook/watchman/blob/f1dcd80d5b1aba75de505dc15a6f7f41092689f0/watchman/query/since.cpp#L66-L76

```cpp
return folly::variant_match(
    since.since,
    [&](const QuerySince::Timestamp& since_ts) -> std::optional<bool> {
      return clock->timestamp >= since_ts.time;
    },
    [&](const QuerySince::Clock& since_clock) -> std::optional<bool> {
      if (since_clock.is_fresh_instance) {
        return file->exists();
      }
      return clock->ticks > since_clock.ticks;
    });
```

```rust
return since.since match -> std::optional<bool> {
  <QuerySince::Timestamp> let since_ts =>
    clock->timestamp >= since_ts.time;
  <QuerySince::Clock> let since_clock => do {
    if (since_clock.is_fresh_instance) {
      do_return file->exists();
    }
    do_return clock->ticks > since_clock.ticks;
  }
};
```

Example 2: https://github.com/facebook/hhvm/blob/c6b07ce3300ed02fd25026b23e6786deb69e88e5/hphp/runtime/ext/thrift/ext_thrift.h#L382-L388

```cpp
folly::variant_match(
    message,
    [&](folly::Try<StreamPayload>& payload) {
      terminated = true;
      if (payload.hasValue()) {
        clientCallback_->onFinalResponse(std::move(payload).value());
      } else {
        clientCallback_->onFinalResponseError(
            std::move(payload).exception());
      }
    },
    [&](int64_t n) { credits += n; });
```

```rust
message match {
  <folly::Try<StreamPayload>> let payload => do {
    terminated = true;
    if (payload.hasValue()) {
      clientCallback_->onFinalResponse(std::move(payload).value());
    } else {
      clientCallback_->onFinalResponseError(
          std::move(payload).exception());
    }
  };
  <int64_t> let n => do { credits += n; };
};
```

Example 3: https://github.com/facebook/openr/blob/8e4c6e553f0314763c1595dd6097dd578d771f1c/openr/decision/Decision.cpp#L183-L203

```cpp
folly::variant_match(
    std::move(maybePub).value(),
    [this](thrift::Publication&& pub) {
      processPublication(std::move(pub));
      // Compute routes with exponential backoff timer if needed
      if (pendingUpdates_.needsRouteUpdate()) {
        rebuildRoutesDebounced_();
      }
    },
    [this](thrift::InitializationEvent&& event) {
      CHECK(event == thrift::InitializationEvent::KVSTORE_SYNCED)
          << fmt::format(
                 "Unexpected initialization event: {}",
                 apache::thrift::util::enumNameSafe(event));

      // Received all initial KvStore publications.
      XLOG(INFO) << "[Initialization] All initial publications are "
                    "received from KvStore.";
      initialKvStoreSynced_ = true;
      triggerInitialBuildRoutes();
    });
```

```rust
std::move(maybePub).value() match {
  <thrift::Publication> let pub => do {
    processPublication(std::move(pub));
    // Compute routes with exponential backoff timer if needed
    if (pendingUpdates_.needsRouteUpdate()) {
      rebuildRoutesDebounced_();
    }
  };
  <thrift::InitializationEvent> let event => do {
    CHECK(event == thrift::InitializationEvent::KVSTORE_SYNCED)
        << fmt::format(
               "Unexpected initialization event: {}",
               apache::thrift::util::enumNameSafe(event));

    // Received all initial KvStore publications.
    XLOG(INFO) << "[Initialization] All initial publications are "
                  "received from KvStore.";
    initialKvStoreSynced_ = true;
    triggerInitialBuildRoutes();
  };
}
```

Example 4: https://github.com/facebook/hhvm/blob/a13020f759b4e74b0f8819ba6ff651606ee47aea/hphp/runtime/ext/thrift/ext_thrift.h#L510-L530

```cpp
auto responseStr = null_string;
auto errorStr = null_string;
uint64_t credits = 0;
folly::variant_match(
    creditsOrFinalResponse_,
    [&](const std::unique_ptr<folly::IOBuf>& finalResponse) {
      responseStr = ioBufToString(*finalResponse);
    },
    [&](const thrift::TClientStreamError& finalResponseError) {
      if (finalResponseError.errorMsg_) {
        // Encoded stream exceptions need to be deserialized by generated
        // code. Returning it as the response string so that it will be
        // decoded
        if (finalResponseError.isEncoded_) {
          responseStr = ioBufToString(*finalResponseError.errorMsg_);
        } else {
          errorStr = ioBufToString(*finalResponseError.errorMsg_);
        }
      }
    },
    [&](const uint64_t& n) { credits = n; });
```

```rust
auto responseStr = null_string;
auto errorStr = null_string;
uint64_t credits = 0;
creditsOrFinalResponse_ match {
  <std::unique_ptr<folly::IOBuf>> let finalResponse => do {
    responseStr = ioBufToString(*finalResponse);
  };
  <thrift::TClientStreamError> (
    let [.errorMsg_: msg_, .isEncoded_: isEncoded]
  ) => do {
    if (msg) {
      // Encoded stream exceptions need to be deserialized by generated
      // code. Returning it as the response string so that it will be
      // decoded
      (isEncoded ? responseStr : errorStr) = ioBufToString(*msg);
    }
  };
  <uint64_t> let n => do { credits = n; };
};
```

Example 5: https://github.com/facebook/fb303/blob/e25ff7e5959dd7624a0e5bc21608d3c949edd269/fb303/ThreadCachedServiceData.h#L679-L685

```cpp
std::array<std::string, N> subkeyStrings;
for (size_t i = 0; i < N; ++i) {
  subkeyStrings[i] = folly::variant_match(
      subkeyArray[i],
      [](int64_t v) { return std::to_string(v); },
      [](std::string const& v) { return v; });
}
```

```rust
std::array<std::string, N> subkeyStrings;
for (size_t i = 0; i < N; ++i) {
  subkeyStrings[i] = subkeyArray[i] match {
    <int64_t> let i => std::to_string(i);
    <std::string> let v => v;
  };
}
```