Example 1: https://github.com/facebook/fboss/blob/5f317f6c5ffd959e5a9cddf7a3a62e89f157a111/fboss/agent/hw/sai/switch/SaiNeighborManager.cpp#L250-L258

```cpp
return std::visit(
    folly::overload(
        [](const std::shared_ptr<ManagedVlanRifNeighbor>& /*handle*/) {
          return cfg::InterfaceType::VLAN;
        },
        [](const std::shared_ptr<PortRifNeighbor>& /*handle*/) {
          return cfg::InterfaceType::SYSTEM_PORT;
        }),
    neighbor_);
```

```rust
return neighbor_ match {
    <std::shared_ptr<ManagedVlanRifNeighbor>> => cfg::InterfaceType::VLAN;
    <std::shared_ptr<PortRifNeighbor>> => cfg::InterfaceType::SYSTEM_PORT;
};
```

Example 2: https://github.com/facebook/sapling/blob/c57f76b9963e050dec8dac7093920f22551b10a1/eden/fs/inodes/overlay/OverlayChecker.cpp#L169-L173

```cpp
ObjectId hash = std::visit(
    folly::overload(
        [](std::shared_ptr<const Tree>& tree) { return tree->getHash(); },
        [](TreeEntry& treeEntry) { return treeEntry.getHash(); }),
    treeOrTreeEntry.value());
```

```rust
ObjectId hash = treeOrTreeEntry.value() match {
    <std::shared_ptr<const Tree>> let tree => tree->getHash();
    <TreeEntry> let treeEntry => treeEntry.getHash();
};
```

Example 3: https://github.com/facebook/sapling/blob/c57f76b9963e050dec8dac7093920f22551b10a1/eden/fs/nfs/Nfsd3.cpp#L1569-L1582

```cpp
return std::visit(
    [](auto&& arg) -> std::string {
      using ArgType = std::decay_t<decltype(arg)>;
      if constexpr (std::is_same_v<ArgType, devicedata3>) {
        // TODO(xavierd): format the specdata3 too.
        return fmt::format(
            ", attr=({})", formatSattr3(arg.dev_attributes).str);
      } else if constexpr (std::is_same_v<ArgType, sattr3>) {
        return fmt::format(", attr=({})", formatSattr3(arg).str);
      } else {
        return "";
      }
    },
    data.v);
```

```rust
return data.v match -> std::string {
    or(
        <devicedata3> [.dev_attributes: let attr],
        <sattr3> let attr
    ) => fmt::format(", attr=({})", formatSattr3(attr).str);
    <auto> => "";
};
```

Example 4: https://github.com/facebook/openr/blob/8e4c6e553f0314763c1595dd6097dd578d771f1c/openr/kvstore/KvStore-inl.h#L161-L162

```cpp
const auto& area = std::visit(
    [](auto&& request) -> AreaId { return request.getArea(); }, kvRequest);
```

```rust
const auto& area =
    kvRequest match -> AreaId { <auto> let request => request.getArea(); };
```

Example 5: https://github.com/facebook/sapling/blob/c57f76b9963e050dec8dac7093920f22551b10a1/eden/fs/config/ParentCommit.cpp#L17-L30

```cpp
std::optional<pid_t> ParentCommit::getInProgressPid() const {
  return std::visit(
      [](auto&& state) -> std::optional<pid_t> {
        using StateType = std::decay_t<decltype(state)>;
        if constexpr (std::is_same_v<
                          StateType,
                          WorkingCopyParentAndCheckedOutRevision>) {
          return std::nullopt;
        } else {
          return state.pid;
        }
      },
      state_);
}
```

```rust
std::optional<pid_t> ParentCommit::getInProgressPid() const {
  return state_ match -> std::optional<pid_t> {
    <WorkingCopyParentAndCheckedOutRevision> => std::nullopt;
    <auto> [.pid: let pid] => pid;
  };
}
```

Example 6: https://github.com/facebook/sapling/blob/c57f76b9963e050dec8dac7093920f22551b10a1/eden/fs/config/ParentCommit.cpp#L56-L69

```cpp
RootId ParentCommit::getWorkingCopyParent() const {
  return std::visit(
      [](auto&& state) -> RootId {
        using StateType = std::decay_t<decltype(state)>;
        if constexpr (std::is_same_v<
                          StateType,
                          WorkingCopyParentAndCheckedOutRevision>) {
          return state.workingCopyParent;
        } else {
          return state.to;
        }
      },
      state_);
```

```rust
RootId ParentCommit::getWorkingCopyParent() const {
  return state_ match {
    <WorkingCopyParentAndCheckedOutRevision> [.workingCopyParent: let wcp] => wcp;
    <auto> [.to: let to] => to;
  };
```

Example 7: 

```cpp
struct LinkMonitor::NetlinkEventProcessor {
  LinkMonitor& lm_;
  explicit NetlinkEventProcessor(LinkMonitor& lm) : lm_(lm) {}

  void
  operator()(fbnl::Link&& link) {
    lm_.processLinkEvent(std::move(link));
  }

  void
  operator()(fbnl::IfAddress&& addr) {
    lm_.processAddressEvent(std::move(addr));
  }

  void
  operator()(fbnl::Neighbor&&) {}

  void
  operator()(fbnl::Rule&&) {}
};

// ...

NetlinkEventProcessor visitor(*this);
while (true) {
  auto maybeEvent = q.get();
  if (maybeEvent.hasError()) {
    break;
  }
  std::visit(visitor, std::move(*maybeEvent));
}
```

```rust
while (true) {
    q.get() match {
        ? <fbnl::Link> let link => do {
            processLinkEvent(std::move(link));
        }
        ? <fbnl::IfAddress> let addr => do {
            processAddressEvent(std::move(addr));
        }
        ? <fbnl::Neighbor> => do {}
        ? <fbnl::Rule> => do {}
        _ => break;
    };
}
```

Example 8: https://github.com/facebook/sapling/blob/c57f76b9963e050dec8dac7093920f22551b10a1/eden/fs/utils/FaultInjector.cpp#L42-L68

```cpp
using RV = ImmediateFuture<Unit>;
return std::visit(
    folly::overload(
        [&](const Unit&) -> RV { return folly::unit; },
        [&](const FaultInjector::Block&) -> RV {
          XLOG(DBG1) << "block fault hit: " << keyClass << ", " << keyValue;
          return addBlockedFault(keyClass, keyValue);
        },
        [&](const FaultInjector::Delay& delay) -> RV {
          XLOG(DBG1) << "delay fault hit: " << keyClass << ", " << keyValue;
          if (delay.error.has_value()) {
            return folly::futures::sleep(delay.duration)
                .defer([error = delay.error.value()](auto&&) {
                  error.throw_exception();
                });
          }
          return folly::futures::sleep(delay.duration);
        },
        [&](const folly::exception_wrapper& error) -> RV {
          XLOG(DBG1) << "error fault hit: " << keyClass << ", " << keyValue;
          return RV{std::move(error)};
        },
        [&](const FaultInjector::Kill&) -> RV {
          XLOG(DBG1) << "kill fault hit: " << keyClass << ", " << keyValue;
          abort();
        }),
    behavior);
```

```rust
return behavior match -> ImmediateFuture<Unit> {
    <Unit> => folly::unit;
    <FaultInjector::Block> => do {
      XLOG(DBG1) << "block fault hit: " << keyClass << ", " << keyValue;
      do_return addBlockedFault(keyClass, keyValue);
    };
    <FaultInjector::Delay> let [.error: err, .duration: dur] => do {
      XLOG(DBG1) << "delay fault hit: " << keyClass << ", " << keyValue;
      do_return err match {
        ? let value => folly::futures::sleep(dur)
            .defer([error = value](auto&&) { error.throw_exception(); });
        _ => folly::futures::sleep(dur);
      };
    };
    <folly::exception_wrapper> let error => do {
      XLOG(DBG1) << "error fault hit: " << keyClass << ", " << keyValue;
      do_return std::move(error);
    };
    <FaultInjector::Kill> => do {
      XLOG(DBG1) << "kill fault hit: " << keyClass << ", " << keyValue;
      abort();
    };
};
```

Example 9: https://github.com/facebook/rocksdb/blob/f6fd4b9dbd15dba36f7e5ad23de407b5c26b1460/db/wide/db_wide_basic_test.cc#L1349-L1354

```cpp
size_t count = 0;
std::visit(
    overload{[&](const std::monostate&) {},
             [&](const Slice&) { count = 1; },
             [&](const WideColumns& columns) { count = columns.size(); }},
    merge_in.existing_value);
```

```rust
size_t count = merge_in.existing_value match {
    <std::monostate> => 0;
    <Slice> => 1;
    <WideColumns> let columns => columns.size();
};
```

Example 10: https://github.com/facebookincubator/velox/blob/409877a050f6d53dc820d2f32db8bc4ef4bf8d7f/velox/dwio/parquet/writer/arrow/PathInternal.cpp#L629-L656

```cpp
PathInfo::Node& node = path_info->path[stack_position - stack_base];
struct {
  IterationResult operator()(NullableNode& node) {
    return node.Run(stack_position, stack_position + 1, context);
  }
  IterationResult operator()(ListNode& node) {
    return node.Run(stack_position, stack_position + 1, context);
  }
  IterationResult operator()(NullableTerminalNode& node) {
    return node.Run(*stack_position, context);
  }
  IterationResult operator()(FixedSizeListNode& node) {
    return node.Run(stack_position, stack_position + 1, context);
  }
  IterationResult operator()(AllPresentTerminalNode& node) {
    return node.Run(*stack_position, context);
  }
  IterationResult operator()(AllNullsTerminalNode& node) {
    return node.Run(*stack_position, context);
  }
  IterationResult operator()(LargeListNode& node) {
    return node.Run(stack_position, stack_position + 1, context);
  }
  ElementRange* stack_position;
  PathWriteContext* context;
} visitor = {stack_position, &context};

IterationResult result = std::visit(visitor, node);
```

```rust
IterationResult result = node match {
  or(
    <NullableNode> let node,
    <ListNode> let node,
    <FixedSizeListNode> let node,
    <LargeListNode> let node
  ) => node.Run(stack_position, stack_position + 1, context);
  or(
    <NullableTerminalNode> let node,
    <AllPresentTerminalNode> let node,
    <AllNullsTerminalNode> let node
  ) => node.Run(*stack_position, context);
};
```