# aux4 ksqldb stream

```beforeAll
aux4 ksqldb query run "DROP STREAM IF EXISTS TEST_AUX4_STREAM DELETE TOPIC" 2>/dev/null || true
aux4 ksqldb query run "DROP STREAM IF EXISTS TEST_AUX4_STREAM_DROP DELETE TOPIC" 2>/dev/null || true
aux4 ksqldb query run "CREATE STREAM TEST_AUX4_STREAM (id INT, name VARCHAR) WITH (KAFKA_TOPIC='test-aux4-stream-topic', VALUE_FORMAT='JSON', PARTITIONS=1)"
aux4 ksqldb query run "CREATE STREAM TEST_AUX4_STREAM_DROP (id INT) WITH (KAFKA_TOPIC='test-aux4-stream-drop-topic', VALUE_FORMAT='JSON', PARTITIONS=1)"
```

```afterAll
aux4 ksqldb query run "DROP STREAM IF EXISTS TEST_AUX4_STREAM DELETE TOPIC" 2>/dev/null || true
aux4 ksqldb query run "DROP STREAM IF EXISTS TEST_AUX4_STREAM_DROP DELETE TOPIC" 2>/dev/null || true
```

## list streams

```execute
aux4 ksqldb stream list | jq -r '.[].name' | grep TEST_AUX4_STREAM
```

```expect:partial
TEST_AUX4_STREAM
```

## describe stream

```execute
aux4 ksqldb stream describe TEST_AUX4_STREAM | jq -r '.[0].sourceDescription.name'
```

```expect
TEST_AUX4_STREAM
```

## insert into stream

```execute
aux4 ksqldb query run "INSERT INTO TEST_AUX4_STREAM (id, name) VALUES (1, 'alice')"
```

```expect
[]
```

## insert another row

```execute
aux4 ksqldb query run "INSERT INTO TEST_AUX4_STREAM (id, name) VALUES (2, 'bob')"
```

```expect
[]
```

## insert with params

```execute
aux4 ksqldb query run "INSERT INTO TEST_AUX4_STREAM (id, name) VALUES (3, :name)" --name "charlie"
```

```expect
[]
```

## drop stream

```execute
aux4 ksqldb stream drop TEST_AUX4_STREAM_DROP | jq -r '.[0].commandStatus.status'
```

```expect
SUCCESS
```
