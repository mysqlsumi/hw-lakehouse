# HeatWave-lakehouse
How to run HeatWave Lakehouse

## 1. 실행 환경
### 1) Compute OS : OL 8 on OCI for bastion server
### 2) MDS Shape : MySQL.HeatWave.VM.Standard.E3, enabled lakehouse 
### 3) MySQL version: 8.0.34

## 2. 실행 코드
### 1) sql 실행 명령어 (mysqlsh)
#### 1-1. Using CSV file, Using PAR, Auto Parallel Load
```
create database mydemo;
use mydemo;

SET @db_list = '["mydemo"]';
SET @par = 'https://idazzjlcjqzj.objectstorage.ap-seoul-1.oci.customer-oci.com/n/idazzjlcjqzj/b/iot-csv/o/iot_telemetry_data02.csv';

SET @ext_tables = '[{"db_name": "mydemo", "tables": [{"table_name": "iot_data", "dialect": {"format": "csv", "skip_rows": 1,  "date_format": "auto", "time_format": "auto","trim_spaces": true,  "field_delimiter": ",","record_delimiter": "\\n"}, "file": [{"par": "https://objectstorage.us-ashburn-1.oraclecloud.com/p/Pgz9ZJ91fRQs0Wlnzkr-eZnhSz7BGAsVeWwhEigna42PuWTOD7oQECjCvprJcAAs/n/idazzjlcjqzj/b/HWdatalake/o/airlines.csv" }] }] }]';

SET @options = JSON_OBJECT('mode', 'normal', 'external_tables', CAST(@ext_tables AS JSON));
call sys.heatwave_load(@db_list, @options);
SELECT * FROM mydemo.iot_data LIMIT3;
SELECT log->>"$.sql" AS "Load Script" FROM sys.heatwave_autopilot_report WHERE type = "sql" ORDER BY id\G
```
#### 1-2. Using CSV file, Using PAR, manual laod
```
CREATE TABLE IF NOT EXISTS `iot_data` (   `co` double,   `humidity` double ,   `temp` double  ) ENGINE=lakehouse SECONDARY_ENGINE=rapid   ENGINE_ATTRIBUTE='{"dialect": {"format": "csv", "field_delimiter":",", "record_delimiter": "\\n"},     "file": [{"par": "https://objectstorage.ap-seoul-1.oraclecloud.com/n/idazzjlcjqzj/b/iot-csv/o/iot_dataiot_telemetry_data3.csv"}]}';

ALTER TABLE hwdemo.supplier SECONDARY_LOAD;
SELECT * FROM mydemo.iot_data LIMIT3;
```

#### 1-3. Using TSV file, Using Resource Principal, manual laod
In this case, you have to create a dynamic group before you run the commands. With our internal reason, I can not create a dynamic group, so I just add it that I received from Yamasaki-san.
```

CREATE TABLE IF NOT EXISTS `supplier` (
  `S_SUPPKEY` int NOT NULL,
  `S_NAME` char(25) COLLATE utf8mb4_bin NOT NULL,
  `S_ADDRESS` varchar(40) COLLATE utf8mb4_bin NOT NULL,
  `S_NATIONKEY` int NOT NULL,
  `S_PHONE` char(15) COLLATE utf8mb4_bin NOT NULL,
  `S_ACCTBAL` decimal(15,2) NOT NULL,
  `S_COMMENT` varchar(101) COLLATE utf8mb4_bin NOT NULL,
  PRIMARY KEY (`S_SUPPKEY`)
) ENGINE=lakehouse SECONDARY_ENGINE=rapid
  ENGINE_ATTRIBUTE='{"dialect": {"format": "csv", "field_delimiter": "\\t", "record_delimiter": "\\n"},
    "file": [{"region": "ap-osaka-1", "namespace": "idazzjlcjqzj", "bucket": "Lakehouse", "name": "tpch/supplier0.tsv"}]}';

ALTER TABLE tpch2.supplier SECONDARY_LOAD;

SELECT * FROM tpch2.supplier LIMIT 1;
```

### 2) 실행 결과
```
1-1 실행결과

 MySQL  10.0.1.59:33060+ ssl  mydemo  SQL > SET @db_list = '["mydemo"]';
Query OK, 0 rows affected (0.0009 sec)
 MySQL  10.0.1.59:33060+ ssl  mydemo  SQL > SET @par = 'https://objectstorage.ap-seoul-1.oraclecloud.com/n/idazzjlcjqzj/b/iot-csv/o/iot_dataiot_telemetry_data3.csv';
Query OK, 0 rows affected (0.0003 sec)
 MySQL  10.0.1.59:33060+ ssl  mydemo  SQL > SET @ext_tables = '[{"db_name": "mydemo", "tables": [{"table_name": "iot_data2", "dialect": {"format": "csv", "skip_rows": 1, "date_format": "auto", "time_format": "auto","trim_spaces": true, "field_delimiter": ",","record_delimiter": "\\n"}, "file": [{"par": "https://objectstorage.ap-seoul-1.oraclecloud.com/n/idazzjlcjqzj/b/iot-csv/o/iot_dataiot_telemetry_data3.csv" }] }] }]';
Query OK, 0 rows affected (0.0004 sec)
 MySQL  10.0.1.59:33060+ ssl  mydemo  SQL > SET @options = JSON_OBJECT('mode', 'normal', 'external_tables', CAST(@ext_tables AS JSON));
Query OK, 0 rows affected (0.0004 sec)
 MySQL  10.0.1.59:33060+ ssl  mydemo  SQL > call sys.heatwave_load(@db_list, @options);
+------------------------------------------+
| INITIALIZING HEATWAVE AUTO PARALLEL LOAD |
+------------------------------------------+
| Version: 2.20                            |
|                                          |
| Load Mode: normal                        |
| Load Policy: disable_unsupported_columns |
| Output Mode: normal                      |
|                                          |
+------------------------------------------+
6 rows in set (9.5678 sec)

+---------------------------------------------------------------------------------------------------------------------------------------------------+
| LAKEHOUSE AUTO SCHEMA INFERENCE                                                                                                                   |
+---------------------------------------------------------------------------------------------------------------------------------------------------+
| Verifying external lakehouse tables: 2                                                                                                            |
|                                                                                                                                                   |
| SCHEMA                   TABLE                    TABLE IS           RAW     NUM. OF      ESTIMATED     SUMMARY OF                                |
| NAME                     NAME                     CREATED      FILE SIZE     COLUMNS      ROW COUNT     ISSUES                                    |
| ------                   -----                    --------     ---------     -------      ---------     ----------                                |
| `mydemo`                 `iot_data`               YES                  -           -              -     Non-empty existing table is not supported |
| `mydemo`                 `iot_data2`              NO           11.02 MiB           3       405.18 K                                               |
|                                                                                                                                                   |
| New schemas to be created: 0                                                                                                                      |
| External lakehouse tables to be created: 1                                                                                                        |
|                                                                                                                                                   |
+---------------------------------------------------------------------------------------------------------------------------------------------------+
11 rows in set (9.5678 sec)

+-----------------------------------------------------------------------------------------+
| OFFLOAD ANALYSIS                                                                        |
+-----------------------------------------------------------------------------------------+
| Verifying input schemas: 1                                                              |
| User excluded items: 0                                                                  |
|                                                                                         |
| SCHEMA                       OFFLOADABLE    OFFLOADABLE     SUMMARY OF                  |
| NAME                              TABLES        COLUMNS     ISSUES                      |
| ------                       -----------    -----------     ----------                  |
| `mydemo`                               1              3     1 table(s) are not loadable |
|                                                                                         |
| Total offloadable schemas: 1                                                            |
|                                                                                         |
+-----------------------------------------------------------------------------------------+
10 rows in set (9.5678 sec)

+-----------------------------------------------------------------------------------------------------------------------------+
| CAPACITY ESTIMATION                                                                                                         |
+-----------------------------------------------------------------------------------------------------------------------------+
| Default encoding for string columns: VARLEN (unless specified in the schema)                                                |
| Estimating memory footprint for 1 schema(s)                                                                                 |
|                                                                                                                             |
|                                TOTAL       ESTIMATED       ESTIMATED       TOTAL     DICTIONARY      VARLEN       ESTIMATED |
| SCHEMA                   OFFLOADABLE   HEATWAVE NODE      MYSQL NODE      STRING        ENCODED     ENCODED            LOAD |
| NAME                          TABLES       FOOTPRINT       FOOTPRINT     COLUMNS        COLUMNS     COLUMNS            TIME |
| ------                   -----------       ---------       ---------     -------     ----------     -------       --------- |
| `mydemo`                           1       10.26 MiB      256.00 KiB           0              0           0          1.00 s |
|                                                                                                                             |
| Sufficient MySQL host memory available to load all tables.                                                                  |
| Sufficient HeatWave cluster memory available to load all tables.                                                            |
|                                                                                                                             |
+-----------------------------------------------------------------------------------------------------------------------------+
12 rows in set (9.5678 sec)

+---------------------------------------------------------------------------------------------------------------------------------------+
| EXECUTING LOAD                                                                                                                        |
+---------------------------------------------------------------------------------------------------------------------------------------+
| HeatWave Load script generated                                                                                                        |
|   Retrieve load script containing 2 generated DDL command(s) using the query below:                                                   |
| Deprecation Notice: "heatwave_load_report" will be deprecated, please switch to "heatwave_autopilot_report"                           |
|   SELECT log->>"$.sql" AS "Load Script" FROM sys.heatwave_autopilot_report WHERE type = "sql" ORDER BY id;                            |
|                                                                                                                                       |
| Adjusting load parallelism dynamically per internal/external table.                                                                   |
| Using current parallelism of 32 thread(s) as maximum for internal tables.                                                             |
|                                                                                                                                       |
| Using SQL_MODE: ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION |
|                                                                                                                                       |
| Proceeding to load 1 table(s) into HeatWave.                                                                                          |
|                                                                                                                                       |
| Applying changes will take approximately 101.14 ms                                                                                    |
|                                                                                                                                       |
+---------------------------------------------------------------------------------------------------------------------------------------+
14 rows in set (9.5678 sec)

+----------------------------------------+
| LOADING TABLE                          |
+----------------------------------------+
| TABLE (1 of 1): `mydemo`.`iot_data2`   |
| Commands executed successfully: 2 of 2 |
| Warnings encountered: 0                |
| Table loaded successfully!             |
|   Total columns loaded: 3              |
|   Elapsed time: 6.13 s                 |
|                                        |
+----------------------------------------+
7 rows in set (9.5678 sec)

+-------------------------------------------------------------------------------+
| LOAD SUMMARY                                                                  |
+-------------------------------------------------------------------------------+
|                                                                               |
| SCHEMA                          TABLES       TABLES      COLUMNS         LOAD |
| NAME                            LOADED       FAILED       LOADED     DURATION |
| ------                          ------       ------      -------     -------- |
| `mydemo`                             1            0            3       6.13 s |
|                                                                               |
+-------------------------------------------------------------------------------+
6 rows in set (9.5678 sec)

Query OK, 0 rows affected (9.5678 sec)
 MySQL  10.0.1.59:33060+ ssl  mydemo  SQL > desc iot_data2;
+-------+---------------+------+-----+---------+-------+
| Field | Type          | Null | Key | Default | Extra |
+-------+---------------+------+-----+---------+-------+
| col_1 | decimal(10,9) | NO   |     | NULL    |       |
| col_2 | decimal(11,9) | NO   |     | NULL    |       |
| col_3 | decimal(11,9) | NO   |     | NULL    |       |
+-------+---------------+------+-----+---------+-------+
3 rows in set (0.0014 sec)
 MySQL  10.0.1.59:33060+ ssl  mydemo  SQL > select * from iot_data2 limit 3;
+-------------+--------------+--------------+
| col_1       | col_2        | col_3        |
+-------------+--------------+--------------+
| 0.004492535 | 53.200000760 | 30.600000380 |
| 0.006059820 | 50.000000000 | 22.600000000 |
| 0.004940912 | 76.800003050 | 19.000000000 |
+-------------+--------------+--------------+
3 rows in set (0.0166 sec)
 MySQL  10.0.1.59:33060+ ssl  mydemo  SQL > SELECT log->>"$.sql" AS "Load Script" FROM sys.heatwave_autopilot_report WHERE type = "sql" ORDER BY id\G
*************************** 1. row ***************************
Load Script: CREATE TABLE `mydemo`.`iot_data2`( `col_1` decimal(10,9) NOT NULL, `col_2` decimal(11,9) NOT NULL, `col_3` decimal(11,9) NOT NULL) ENGINE=lakehouse SECONDARY_ENGINE=RAPID ENGINE_ATTRIBUTE='{"file": [{"par": "https://objectstorage.ap-seoul-1.oraclecloud.com/n/idazzjlcjqzj/b/iot-csv/o/iot_dataiot_telemetry_data3.csv"}], "dialect": {"format": "csv", "skip_rows": 1, "date_format": "auto", "time_format": "auto", "trim_spaces": true, "field_delimiter": ",", "record_delimiter": "\\n"}}';
*************************** 2. row ***************************
Load Script: ALTER TABLE `mydemo`.`iot_data2` SECONDARY_LOAD;
2 rows in set (0.0006 sec)
 MySQL  10.0.1.59:33060+ ssl  mydemo  SQL > 


1-2 실행 결과

 MySQL  10.0.1.59:33060+ ssl  hwdemo  SQL > CREATE TABLE IF NOT EXISTS `supplier` (   `co` double,   `humidity` double ,   `temp` double  ) ENGINE=lakehouse SECONDARY_ENGINE=rapid   ENGINE_ATTRIBUTE='{"dialect": {"format": "csv", "field_delimiter":",", "record_delimiter": "\\n"},     "file": [{"par": "https://objectstorage.ap-seoul-1.oraclecloud.com/n/idazzjlcjqzj/b/iot-csv/o/iot_dataiot_telemetry_data3.csv"}]}';

Query OK, 0 rows affected (0.0053 sec)

 MySQL  10.0.1.59:33060+ ssl  hwdemo  SQL > show tables;

+------------------+

| Tables_in_hwdemo |

+------------------+

| supplier         |

+------------------+

1 row in set (0.0010 sec)

 MySQL  10.0.1.59:33060+ ssl  hwdemo  SQL > desc supplier;

+----------+--------+------+-----+---------+-------+

| Field    | Type   | Null | Key | Default | Extra |

+----------+--------+------+-----+---------+-------+

| co       | double | YES  |     | NULL    |       |

| humidity | double | YES  |     | NULL    |       |

| temp     | double | YES  |     | NULL    |       |

+----------+--------+------+-----+---------+-------+

3 rows in set (0.0017 sec)

 MySQL  10.0.1.59:33060+ ssl  hwdemo  SQL > ALTER TABLE hwdemo.supplier SECONDARY_LOAD;

Query OK, 0 rows affected, 4 warnings (6.4117 sec)

Warning (code 3877): Col 1 of row starting at {object: iot_dataiot_telemetry_data3.csv in region: ap-seoul-1, namespace: idazzjlcjqzj, bucket: iot-csv}:0: Data was truncated

Warning (code 3877): Col 2 of row starting at {object: iot_dataiot_telemetry_data3.csv in region: ap-seoul-1, namespace: idazzjlcjqzj, bucket: iot-csv}:0: Data was truncated

Warning (code 3877): Col 3 of row starting at {object: iot_dataiot_telemetry_data3.csv in region: ap-seoul-1, namespace: idazzjlcjqzj, bucket: iot-csv}:0: Data was truncated

Warning (code 3877): Total Warnings: 3

```

## 3. 참고자료
- [Oracle blog](https://blogs.oracle.com/mysql/post/getting-started-with-mysql-heatwave-lakehouse)
- [MySQL ref menual](https://dev.mysql.com/doc/heatwave/en/mys-hw-lakehouse.html)


