# aux4 ksqldb topic

```beforeAll
aux4 ksqldb query run "DROP STREAM IF EXISTS TEST_AUX4_TOPIC_STREAM DELETE TOPIC" 2>/dev/null || true
aux4 ksqldb query run "CREATE STREAM TEST_AUX4_TOPIC_STREAM (id INT) WITH (KAFKA_TOPIC='test-aux4-topic', VALUE_FORMAT='JSON', PARTITIONS=1)"
```

```afterAll
aux4 ksqldb query run "DROP STREAM IF EXISTS TEST_AUX4_TOPIC_STREAM DELETE TOPIC" 2>/dev/null || true
```

## list topics

```execute
aux4 ksqldb topic list | jq -r '.[].name' | grep test-aux4-topic
```

```expect
test-aux4-topic
```

