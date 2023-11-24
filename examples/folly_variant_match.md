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
  <QuerySince::Clock> let since_clock => 
    since_clock.is_fresh_instance
      ? file->exists()
      : clock->ticks > since_clock.ticks;
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

