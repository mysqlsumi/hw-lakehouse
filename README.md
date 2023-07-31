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
In this case, you have to create a dynamic resource before you run the commands.
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

### 2) 실행 캡쳐 화면
<img width="1131" alt="image" src="https://github.com/mysqlsumi/hw-lakehouse/assets/31054795/d988870a-742d-43c3-b750-0406c3b7fc53">
<img width="1131" alt="image" src="https://github.com/mysqlsumi/hw-lakehouse/assets/31054795/93dcc27a-e3e3-4170-8318-93791137b60d">


## 3. 참고자료
- [Oracle blog](https://blogs.oracle.com/mysql/post/getting-started-with-mysql-heatwave-lakehouse)
- [MySQL ref menual](https://dev.mysql.com/doc/heatwave/en/mys-hw-lakehouse.html)


