---
title: Log user and system events
prefix: Logging
---

[TOC]

To log any user or system event, you can call `gramex.log(key=value, key=value, ...)`. This logs
into an ElasticSearch database that you can configure.

[**First**, install ElasticSearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/install-elasticsearch.html)
on any server, or get a [cloud](https://www.elastic.co/cloud/) instance.

**Second**, add this to `gramex.yaml`:

```yaml
gramexlog:
  gramexguide:              # Log name. We'll call this via gramex.log('gramexguide')
    host: hostname          # OPTIONAL: ElasticSearch server name. default: localhost
    port: 9200              # OPTIONAL: port to connect to. default: 9200
    index: gramexguide      # OPTIONAL: index to connect to. default: same as log name
    http_auth: user:pass    # OPTIONAL: user name and password. default: None
    keys: [datetime, user.id, headers.User-Agent]   # OPTIONAL: additional keys. default: None
```

In the `keys:` section, you can use any key supported by the `log:` service
[documented here](../config/#request-logging). Every log record will automatically add these keys.

**Finally**, call `gramex.log(x=1, y=2)` (or use any other keyword arguments.)

This will log `{"port": 9988, "time": ..., "user.id": ..., "x": 1, "y": 2}` into your ElasticSearch
server's index `gramexguide`.

## Logging handler

You can use `gramex.log` as a [FunctionHandler](../functionhandler/) endpoint directly.

```yaml
url:
  log:
    pattern: /log
    handler: FunctionHandler
    kwargs:
      function: gramex.log(handler, 'gramexguide', event='handler')
```

When a user visit `/log?x=1&y=2`, it logs into `gramexguide` an object with these keys:

- `"x": "1"` from the URL
- `"y": "2"` from the URL
- `"event": "handler"` from the function call
- `"datetime": ...` from the `gramexguide` index configuration
- `"user.id": ...` from the `gramexguide` index configuration
- `"header.User-Agent": ...` from the `gramexguide` index configuration

<div class="example">
  <a class="example-demo" href="log">Log an event (result should be null)</a>
  <a class="example-src" href="https://github.com/gramener/gramex-guide/blob/master/logging/gramex.yaml">Source</a>
</div>


## Viewing logs

To access an index in ElasticSearch, use [FormHandler](../formhandler/) after running
`pip install elasticsearch-dbapi`. Use this config:

```yaml
    kwargs:
      url: elasticsearch+http://localhost:9200
      table: gramexguide
```

<div class="example">
  <a class="example-demo" href="view?_format=html&_sort=-datetime">View ElasticSearch log</a>
  <a class="example-src" href="https://github.com/gramener/gramex-guide/blob/master/logging/gramex.yaml">Source</a>
</div>

Apart from FormHandler, you can also use:

- `gramex.data.filter('elasticsearch+http://localhost:9200', table='...', args={...})`.
- [`elasticsearch`](https://elasticsearch-py.readthedocs.io/en/master/) Python client via
`pip install elasticsearch`, and use:

```python
from elasticsearch import Elasticsearch
es = Elasticsearch()
indices = es.indices.get('*')         # Lists indices
records = es.search(index='<index>')  # Gets records from index
```

ElasticSearch also supports a REST API. You can visit

- `/_cat/indices` lists indices
- `/<index>/search` gets records from index


## Multiple logs

You can create multiple logs -- to the same or different server. For example:

```yaml
gramexlog:
  log1:               # gramex.log('log1') logs to localhost:9200 'log1' index
    keys: [time]                # ... with time always added as a key
  log2:               # gramex.log(handler, 'log2') also logs to localhost:9200
    index: log1                 # ... and the same 'log1' index
    keys: [datetime, user.id]   # ... but with datetime and user.id as keys
  log3:               # gramex.log('log3') logs to server.com:9200
    host: server.com
    http_auth: user:pass        # ... using the specified user ID and password
```
