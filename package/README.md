# community/ksqldb

Commands to interact with ksqlDB using the REST API

This package provides a small set of aux4 commands for managing ksqlDB and Kafka objects via the ksqlDB REST API. Use it to query server info and health, list and manage topics, streams, and tables, and to run ad-hoc ksql statements and pull/push queries. It’s designed to be a lightweight CLI wrapper around curl + jq so you can script common ksqlDB tasks from the terminal and in CI.

## Installation

```bash
aux4 aux4 pkger install community/ksqldb
```

## System Dependencies

This package requires system dependencies. You need to have one of the following system installers:

- [brew](/r/public/packages/aux4/system-installer-brew)

For more details, see [system-installer](/r/public/packages/aux4/pkger/commands/aux4/pkger/system).

## Quick Start

Run a quick check against a local ksqlDB server (defaults to http://localhost:8088):

```bash
aux4 ksqldb info
```

This queries the server info endpoint and prints the raw JSON response from ksqlDB. In tests the server status is read like this:

```bash
aux4 ksqldb info | jq -r '.KsqlServerInfo.serverStatus'
```

which returns RUNNING for a healthy server.

## Server: info & health

Get server basic information and health status. Both commands accept the optional variables server, username and password (server defaults to http://localhost:8088).

- Get server info: [aux4 ksqldb info](./commands/ksqldb/info)

Example:

```bash
aux4 ksqldb info | jq -r '.KsqlServerInfo.serverStatus'
```

This prints the ksqlDB server status (for example: RUNNING).

- Check server health: [aux4 ksqldb health](./commands/ksqldb/health)

Example:

```bash
aux4 ksqldb health | jq -r '.isHealthy'
```

This returns true when the server reports healthy.

## Topics

List Kafka topics visible to ksqlDB.

- List topics: [aux4 ksqldb topic list](./commands/ksqldb/topic/list)

Example (from tests):

```bash
aux4 ksqldb topic list | jq -r '.[].name' | grep test-aux4-topic
```

This looks for the test topic name `test-aux4-topic` in the listed topics.

## Streams

Manage and inspect ksqlDB streams.

- List streams: [aux4 ksqldb stream list](./commands/ksqldb/stream/list)
- Describe a stream: [aux4 ksqldb stream describe](./commands/ksqldb/stream/describe)
- Drop a stream: [aux4 ksqldb stream drop](./commands/ksqldb/stream/drop)

Examples (extracted from tests):

List and find a test stream:

```bash
aux4 ksqldb stream list | jq -r '.[].name' | grep TEST_AUX4_STREAM
```

Describe a specific stream:

```bash
aux4 ksqldb stream describe TEST_AUX4_STREAM | jq -r '.[0].sourceDescription.name'
```

Drop a stream that was created for tests:

```bash
aux4 ksqldb stream drop TEST_AUX4_STREAM_DROP | jq -r '.[0].commandStatus.status'
```

The last command returns SUCCESS when the drop completed.

## Tables

List, describe and drop materialized tables.

- List tables: [aux4 ksqldb table list](./commands/ksqldb/table/list)
- Describe a table: [aux4 ksqldb table describe](./commands/ksqldb/table/describe)
- Drop a table: [aux4 ksqldb table drop](./commands/ksqldb/table/drop)

Examples (from tests):

List and match a test table:

```bash
aux4 ksqldb table list | jq -r '.[].name' | grep TEST_AUX4_TABLE
```

Describe the table:

```bash
aux4 ksqldb table describe TEST_AUX4_TABLE | jq -r '.[0].sourceDescription.name'
```

Drop a test table and check status:

```bash
aux4 ksqldb table drop TEST_AUX4_TABLE_DROP | jq -r '.[0].commandStatus.status'
```

## Queries

Run ksql statements, pull queries, push queries, and manage running queries.

- List running queries: [aux4 ksqldb query list](./commands/ksqldb/query/list)
- Run a statement (DDL/DML): [aux4 ksqldb query run](./commands/ksqldb/query/run)
- Execute a pull (SELECT) query: [aux4 ksqldb query select](./commands/ksqldb/query/select)
- Execute a push (EMIT CHANGES) query: [aux4 ksqldb query push](./commands/ksqldb/query/push)
- Terminate a query: [aux4 ksqldb query terminate](./commands/ksqldb/query/terminate)
- Explain a query plan: [aux4 ksqldb query explain](./commands/ksqldb/query/explain)

Examples (from tests):

List queries (returns an array):

```bash
aux4 ksqldb query list | jq -r 'type'
```

Run statements to show topics/streams/tables (returns typed JSON array entries). For example:

```bash
aux4 ksqldb query run "show topics" | jq -r '.[0]["@type"]'
```

which prints kafka_topics.

Create a stream/table and perform insert + select (tests perform setup/teardown around these commands). To insert a row and then query the materialized table (the test waits briefly to allow the table to be populated):

```bash
aux4 ksqldb query run "INSERT INTO TEST_AUX4_QUERY_STREAM (id, name) VALUES (1, 'test')" > /dev/null && sleep 8 && aux4 ksqldb query select "SELECT * FROM TEST_AUX4_QUERY_TABLE WHERE id = 1" | head -1 | jq -r '.columnNames[0]'
```

This sequence inserts a row into the stream, waits for the table to be populated, and then prints the first column name (test expectation: ID).

## Examples

### Check server status

This example reads the server info and prints the serverStatus field:

```bash
aux4 ksqldb info | jq -r '.KsqlServerInfo.serverStatus'
```

In tests this returns:

```
RUNNING
```

### List topics and find a test topic

This example lists topics and filters for the test topic created by the test suite:

```bash
aux4 ksqldb topic list | jq -r '.[].name' | grep test-aux4-topic
```

Expected line in tests:

```
test-aux4-topic
```

### Create/inspect streams (tests use query run to create)

The test suite creates streams via query.run, then uses the stream list and describe commands:

```bash
aux4 ksqldb stream list | jq -r '.[].name' | grep TEST_AUX4_STREAM
```

and

```bash
aux4 ksqldb stream describe TEST_AUX4_STREAM | jq -r '.[0].sourceDescription.name'
```

Both return the stream name when successful.

### Insert into stream and read from materialized table

A test inserts into a stream and then executes a pull query against the derived table to verify the data:

```bash
aux4 ksqldb query run "INSERT INTO TEST_AUX4_QUERY_STREAM (id, name) VALUES (1, 'test')" > /dev/null && sleep 8 && aux4 ksqldb query select "SELECT * FROM TEST_AUX4_QUERY_TABLE WHERE id = 1" | head -1 | jq -r '.columnNames[0]'
```

This prints the column name (expected in tests: ID) once the row is materialized.

## Troubleshooting

- If commands fail, confirm the server variable is pointing to a running ksqlDB REST API (default: http://localhost:8088).
- If your ksqlDB requires basic auth, pass username and password variables when invoking commands (both username and password are optional and default to empty).
- Commands use curl and jq; ensure both are installed (brew package names: curl and jq).

## License

This package does not specify a license. See [LICENSE](./license) for details.
