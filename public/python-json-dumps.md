---
title: Python の json.dumps() で TypeError になる場合の対処方法
tags:
  - Python
private: false
updated_at: '2024-10-16T01:03:31+09:00'
id: 63dc55be836eec4078ee
organization_url_name: null
slide: false
ignorePublish: false
---

## 課題

Python の `json.dumps()` を使うと TypeError になる。

```text
Traceback (most recent call last):
  File "/home/takuya/json_test/main.py", line 32, in <module>
    json.dumps(message)
  File "/usr/lib/python3.8/json/__init__.py", line 231, in dumps
    return _default_encoder.encode(obj)
  File "/usr/lib/python3.8/json/encoder.py", line 199, in encode
    chunks = self.iterencode(o, _one_shot=True)
  File "/usr/lib/python3.8/json/encoder.py", line 257, in iterencode
    return _iterencode(o, 0)
  File "/usr/lib/python3.8/json/encoder.py", line 179, in default
    raise TypeError(f'Object of type {o.__class__.__name__} '
TypeError: Object of type datetime is not JSON serializable
```

## 対策

`json.dumps()` の引数 `default` に、変換できない型が来た場合の処理を書いた関数を渡せばよい。これはネストされたオブジェクトに対しても機能する。

### プログラム例

```python
import json
import datetime

message = {
    "log_level": "ERROR",
    "time_stamp": datetime.datetime.now(),
    "message": "Unexpected Error: Check detail.",
    "detail": {
        "exec_mode": "strict",
        "state": {
            "start": b"some state",
            "end": b"another state",
        },
    },
}


def custom_encode(obj):
    if isinstance(obj, datetime.date):
        return obj.isoformat()

    if isinstance(obj, bytes):
        return obj.decode()

    raise TypeError(f"Cannot serialize object of {type(obj)}")


def json_dumps(obj):
    return json.dumps(obj, indent=2, default=custom_encode)


print(json_dumps(message))

```

### 実行結果

```text
{
  "log_level": "ERROR",
  "time_stamp": "2024-10-16T00:58:49.209903",
  "message": "Unexpected Error: Check detail.",
  "detail": {
    "exec_mode": "strict",
    "state": {
      "start": "some state",
      "end": "another state"
    }
  }
}
```
