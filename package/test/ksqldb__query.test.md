# aux4 ksqldb query

```beforeAll
aux4 ksqldb query run "DROP TABLE IF EXISTS TEST_AUX4_QUERY_TABLE DELETE TOPIC" 2>/dev/null || true
aux4 ksqldb query run "DROP STREAM IF EXISTS TEST_AUX4_QUERY_STREAM DELETE TOPIC" 2>/dev/null || true
aux4 ksqldb query run "CREATE STREAM TEST_AUX4_QUERY_STREAM (id INT KEY, name VARCHAR) WITH (KAFKA_TOPIC='test-aux4-query-stream-topic', VALUE_FORMAT='JSON', PARTITIONS=1)"
aux4 ksqldb query run "CREATE TABLE TEST_AUX4_QUERY_TABLE AS SELECT id, LATEST_BY_OFFSET(name) as name FROM TEST_AUX4_QUERY_STREAM GROUP BY id"
```

```afterAll
aux4 ksqldb query run "DROP TABLE IF EXISTS TEST_AUX4_QUERY_TABLE DELETE TOPIC" 2>/dev/null || true
aux4 ksqldb query run "DROP STREAM IF EXISTS TEST_AUX4_QUERY_STREAM DELETE TOPIC" 2>/dev/null || true
```

## list queries

```execute
aux4 ksqldb query list | jq -r 'type'
```

```expect
array
```

## run query show topics

```execute
aux4 ksqldb query run "show topics" | jq -r '.[0]["@type"]'
```

```expect
kafka_topics
```

## run query show streams

```execute
aux4 ksqldb query run "show streams" | jq -r '.[0]["@type"]'
```

```expect
streams
```

## run query show tables

```execute
aux4 ksqldb query run "show tables" | jq -r '.[0]["@type"]'
```

```expect
tables
```

## explain query

```execute
aux4 ksqldb query list | jq -r '.[0].id' | xargs -I {} aux4 ksqldb query explain {} | jq -r '.[0]["@type"]'
```

```expect
queryDescription
```

## select from table

```timeout
15000
```

```execute
aux4 ksqldb query run "INSERT INTO TEST_AUX4_QUERY_STREAM (id, name) VALUES (1, 'test')" > /dev/null && sleep 8 && aux4 ksqldb query select "SELECT * FROM TEST_AUX4_QUERY_TABLE WHERE id = 1" | head -1 | jq -r '.columnNames[0]'
```

```expect
ID
```
