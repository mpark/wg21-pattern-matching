Example 1: https://github.com/facebook/hhvm/blob/e8e9028007807a87993c3a852d001f2876ccad01/hphp/runtime/ext/string/ext_string.cpp#L2499-L2507

```cpp
for (const auto& item: *get_multicode_table()) {
  auto codes = item.first;
  String key = encode_as_utf8(codes.first);
  key += encode_as_utf8(codes.second);

  char buffer[32];
  snprintf(buffer, sizeof(buffer), "&%s", item.second.c_str());
  ret.set(key, String(buffer, CopyString));
}
```

```cpp
for (const auto& [[code1, code2], value] : *get_multicode_table()) {
  String key = encode_as_utf8(code1);
  key += encode_as_utf8(code2);

  char buffer[32];
  snprintf(buffer, sizeof(buffer), "&%s", value.c_str());
  ret.set(key, String(buffer, CopyString));
}
```

Example 2: https://github.com/facebook/hermes/blob/d90983683580cead28f6e53f5103dde16f5b530b/lib/VM/Runtime.cpp#L758-L762

```cpp
for (const auto &p : arraySizeToCountAndWastedSlots) {
  totalBytes += p.first.second * p.second.first;
  totalCount += p.second.first;
  totalWastedSlots += p.second.second;
}
```

```cpp
for (const auto& [[_, size], [count, wasted]] : arraySizeToCountAndWastedSlots) {
  totalBytes += size * count;
  totalCount += count;
  totalWastedSlots += wasted;
}
```

Example 3: https://github.com/facebook/rocksdb/blob/f6fd4b9dbd15dba36f7e5ad23de407b5c26b1460/util/file_checksum_helper.cc#L34-L38

```cpp
for (auto i : checksum_map_) {
  file_numbers->push_back(i.first);
  checksums->push_back(i.second.first);
  checksum_func_names->push_back(i.second.second);
}
```

```cpp
for (auto [file_num, [checksum, name]] : checksum_map_) {
  file_numbers->push_back(file_num);
  checksums->push_back(checksum);
  checksum_func_names->push_back(name);
}
```