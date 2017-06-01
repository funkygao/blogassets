---
title: TiDB KV Mapping
date: 2017-06-01 14:30:58
tags: database
---

Data
```
Key： tablePrefix_rowPrefix_tableID_rowID
Value: [col1, col2, col3, col4]
```

Unique Index
```
Key: tablePrefix_idxPrefix_tableID_indexID_indexColumnsValue
Value: rowID
```

Non-Unique Index
```
Key: tablePrefix_idxPrefix_tableID_indexID_ColumnsValue_rowID
Value: nil
```

```
var (
    tablePrefix     = []byte{'t'}
    recordPrefixSep = []byte("_r")
    indexPrefixSep  = []byte("_i")
)
```
