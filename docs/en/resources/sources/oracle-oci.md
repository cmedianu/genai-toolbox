---
title: "Oracle OCI"
type: docs
weight: 2
description: >
  Oracle Database connection using Oracle Call Interface (OCI) via godror driver.
---

## About

The `oracle-oci` source provides Oracle Database connectivity using the [godror driver][godror-docs], which wraps the Oracle Call Interface (OCI). Use this source for **Oracle Autonomous Database**, **TNS aliases**, or **Oracle Wallet** authentication. For simple direct connections, use the [`oracle`](../oracle.md) source instead (pure Go, no external dependencies).

[godror-docs]: https://godror.github.io/godror/

## Available Tools

- [`oracle-sql`](../tools/oracle/oracle-sql.md)
  Execute pre-defined prepared SQL queries in Oracle.

- [`oracle-execute-sql`](../tools/oracle/oracle-execute-sql.md)
  Run parameterized SQL queries in Oracle.

## Requirements

### Oracle Instant Client

Requires [Oracle Instant Client][instant-client] (Basic or Basic Light package). Set `LD_LIBRARY_PATH` to the installation directory.

[instant-client]: https://www.oracle.com/database/technologies/instant-client/downloads.html

### Database User

You will need to [create an Oracle user][oracle-users] with necessary permissions.

[oracle-users]: https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/CREATE-USER.html

### Build Requirements

Build with `-tags oracleoci` flag (requires CGO):

```bash
export LD_LIBRARY_PATH=/path/to/instantclient:$LD_LIBRARY_PATH
export CGO_ENABLED=1
go build -tags oracleoci -o toolbox
```

## Connection Methods

You can configure the connection using one of three methods. **Use only one method** in your source configuration.

{{< notice important >}}
**Oracle Wallet / Autonomous Database:** Use TNS Alias method only. Connection String and Host+Port+ServiceName do not support wallet-based authentication.
{{< /notice >}}

### TNS Alias

For environments using `tnsnames.ora` configuration (required for Oracle Autonomous Database and Oracle Wallet):

- `tnsAlias`: Alias name from your `tnsnames.ora` file.
- `tnsAdmin` (Optional): Path to directory containing `tnsnames.ora` and wallet files. Overrides `TNS_ADMIN` environment variable.

### Connection String

Provide all connection details in a single `connectionString`. Format: `hostname:port/servicename`.

### Host + Port + ServiceName

Provide connection details as separate fields:

- `host`: IP address or hostname of the database server.
- `port`: Port number (typically 1521).
- `serviceName`: Service name for the database instance.

## Example

```yaml
sources:
    my-oracle-oci-source:
        kind: oracle-oci
        # --- Choose one connection method ---
        # 1. TNS Alias (required for Autonomous Database / Oracle Wallet)
        tnsAlias: "mydb_high"
        tnsAdmin: "/home/user/wallet/Wallet_MyDB"  # Optional: overrides TNS_ADMIN env var

        # 2. Direct Connection String
        connectionString: "127.0.0.1:1521/XEPDB1"

        # 3. Host, Port, and Service Name
        host: 127.0.0.1
        port: 1521
        serviceName: XEPDB1

        user: ${USER_NAME}
        password: ${PASSWORD}

```

{{< notice tip >}}
Use environment variable replacement with the format ${ENV_NAME}
instead of hardcoding your secrets into the configuration file.
{{< /notice >}}

## Reference

| **field**        | **type** | **required** | **description**                                                                                                             |
|------------------|:--------:|:------------:|-----------------------------------------------------------------------------------------------------------------------------|
| kind             |  string  |     true     | Must be "oracle-oci".                                                                                                       |
| user             |  string  |     true     | Name of the Oracle user to connect as (e.g. "my-oracle-user").                                                              |
| password         |  string  |     true     | Password of the Oracle user (e.g. "my-password").                                                                           |
| host             |  string  |    false     | IP address or hostname to connect to (e.g. "127.0.0.1"). Required if not using `connectionString` or `tnsAlias`.            |
| port             | integer  |    false     | Port to connect to (e.g. "1521"). Defaults to 1521 if not specified.                                                        |
| serviceName      |  string  |    false     | The Oracle service name of the database to connect to. Required if not using `connectionString` or `tnsAlias`.              |
| connectionString |  string  |    false     | A direct connection string (e.g. "hostname:port/servicename"). Use as an alternative to `host`, `port`, and `serviceName`.  |
| tnsAlias         |  string  |    false     | A TNS alias from a `tnsnames.ora` file. Use as an alternative to `host`/`port` or `connectionString`.                       |
| tnsAdmin         |  string  |    false     | Path to the directory containing the `tnsnames.ora` file and wallet. Overrides the `TNS_ADMIN` environment variable.        |

## Comparison: oracle vs oracle-oci

| Feature                  | oracle (go-ora)     | oracle-oci (godror)    |
|--------------------------|---------------------|------------------------|
| External Dependencies    | None                | Oracle Instant Client  |
| Build Tags Required      | No                  | Yes (`-tags oracleoci`)|
| TNS Alias Support        | No                  | Yes                    |
| Oracle Wallet Support    | No                  | Yes                    |
| Autonomous Database      | No                  | Yes                    |
| Pure Go                  | Yes                 | No (uses CGO)          |
