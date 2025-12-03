# aux4 ksqldb table

```timeout
15000
```

```beforeAll
aux4 ksqldb query run "DROP TABLE IF EXISTS TEST_AUX4_TABLE DELETE TOPIC" 2>/dev/null || true
aux4 ksqldb query run "DROP TABLE IF EXISTS TEST_AUX4_TABLE_DROP DELETE TOPIC" 2>/dev/null || true
aux4 ksqldb query run "CREATE TABLE TEST_AUX4_TABLE (id INT PRIMARY KEY, name VARCHAR) WITH (KAFKA_TOPIC='test-aux4-table-topic', VALUE_FORMAT='JSON', PARTITIONS=1)"
aux4 ksqldb query run "CREATE TABLE TEST_AUX4_TABLE_DROP (id INT PRIMARY KEY) WITH (KAFKA_TOPIC='test-aux4-table-drop-topic', VALUE_FORMAT='JSON', PARTITIONS=1)"
```

```afterAll
aux4 ksqldb query run "DROP TABLE IF EXISTS TEST_AUX4_TABLE DELETE TOPIC" 2>/dev/null || true
aux4 ksqldb query run "DROP TABLE IF EXISTS TEST_AUX4_TABLE_DROP DELETE TOPIC" 2>/dev/null || true
```

## list tables

```execute
aux4 ksqldb table list | jq -r '.[].name' | grep TEST_AUX4_TABLE
```

```expect:partial
TEST_AUX4_TABLE
```

## describe table

```execute
aux4 ksqldb table describe TEST_AUX4_TABLE | jq -r '.[0].sourceDescription.name'
```

```expect
TEST_AUX4_TABLE
```

## drop table

```execute
aux4 ksqldb table drop TEST_AUX4_TABLE_DROP | jq -r '.[0].commandStatus.status'
```

```expect
SUCCESS
```

