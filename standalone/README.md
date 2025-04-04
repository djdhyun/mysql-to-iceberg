# Iceberg + Spark + Hive Metastore (PostgreSQL) on Docker

Local development environment for testing **MySQL to Iceberg CDC pipelines**.

## Prerequisites

- Docker & docker-compose (Docker Desktop if Mac)

## Usage

- `make up`: Starts all services
- `make down`: Stops all services
- `make log`: Shows live logs of all services

## Quickstart

1. `make up`
2. `docker ps` to verify the services (spark, hive, postgresql) are running.
3. `../bin/spark-sql` to open a spark-sql session.
4. Run a sample query inside `spark-sql`.

```sql
CREATE NAMESPACE ns1

CREATE TABLE ns1.tbl1 (
  id INT,
  name STRING
) USING iceberg
TBLPROPERTIES ('format-version'='2');

INSERT INTO ns1.tbl1 VALUES (1, 'Alice'), (2, 'Bob');

SELECT * FROM ns1.tbl1;
```

5. `tree warehouse` to inspect the physical data layout on disk.

```
warehouse/
└── ns1.db
    └── tbl1
        ├── data
        │   ├── 00000-0-c1fb2926-2030-4c62-a716-47df60e67698-00001.parquet
        │   └── 00001-1-c1fb2926-2030-4c62-a716-47df60e67698-00001.parquet
        └── metadata
            ├── 00000-0dbe9396-8656-4c10-9b20-e34486d2100b.metadata.json
            ├── 00001-6f1e3509-a1d9-4846-b232-310994d62ed7.metadata.json
            ├── 790eb29b-c8f8-4d62-8411-e7c4afa4ec07-m0.avro
            └── snap-3510122822080613405-1-790eb29b-c8f8-4d62-8411-e7c4afa4ec07.avro
```

6. (Optional) `../bin/psql` to inspect the Hive Metastore using PostgreSQL.
   - (inside the psql session) `SELECT * FROM "TBLS"` will shows table entries.

## Note

- If the hive-metastore service fails to start, it may be due to schema initialization errors. This often happens when the backing PostgreSQL database already contains existing Hive Metastore tables (e.g., TBLS).
  - In such cases, you can run `IS_RESUME=true make up` to start the services without reinitializing the schema.
  - Alternatively, run `make clean && make up` to completely reset all data and start from a clean state.
  - If the problem persists, try running `make down && make up` again.
- Spark, Iceberg, and Hive versions must be compatible! Refer to the [Makefile](./Makefile) for the currently configured versions.
- Although the [official Iceberg documentation claims that Hive 4 supports Iceberg 1.4.3 natively](https://iceberg.apache.org/docs/latest/hive/#hive-400), the `apache/hive:4.0.1` Docker image is not compatible with Iceberg 1.3 or 1.4 in practice.
  - You may encounter errors such as: `org.apache.thrift.TApplicationException: Invalid method name: 'get_table'`.
- Spark 3.5 isn't compatible with Iceberg 1.4 even when using the `iceberg-spark-runtime-3.5_2.12-1.4.3.jar`.
  - `UPDATE` and `MERGE` statements fails with the errors as `java.lang.NoSuchMethodError: 'boolean org.apache.spark.sql.types.DataType$.canWrite`
