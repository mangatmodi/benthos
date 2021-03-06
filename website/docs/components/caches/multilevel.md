---
title: multilevel
type: cache
status: stable
---

<!--
     THIS FILE IS AUTOGENERATED!

     To make changes please edit the contents of:
     lib/cache/multilevel.go
-->

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';


Combines multiple caches as levels, performing read-through and write-through
operations across them.

```yaml
# Config fields, showing default values
multilevel: []
```

For the Add command this cache first checks all levels except the last for the
key. If the key is not found it is added to the final cache level, if that
succeeds all higher cache levels have the key set.

## Examples

It's possible to use multilevel to create a warm cache in memory above a cold
remote cache:

```yaml
pipeline:
  processors:
    - branch:
        processors:
          - cache:
              resource: leveled
              operator: get
              key: ${! json("key") }
          - catch:
            - bloblang: 'root = {"err":error()}'
        result_map: 'root.result = this'

resources:
  caches:
    leveled:
      multilevel: [ hot, cold ]

    hot:
      memory:
        ttl: 300

    cold:
      memcached:
        addresses: [ TODO:11211 ]
        ttl: 3600
```

Using this config when a target key already exists in our local memory cache we
won't bother hitting the remote memcached instance.

