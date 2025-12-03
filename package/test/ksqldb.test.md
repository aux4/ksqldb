# aux4 ksqldb

## info

```execute
aux4 ksqldb info | jq -r '.KsqlServerInfo.serverStatus'
```

```expect
RUNNING
```

## health

```execute
aux4 ksqldb health | jq -r '.isHealthy'
```

```expect
true
```
