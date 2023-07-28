# HeatWave-lakehouse
How to run HeatWave Lakehouse

## 1. 실행 환경
### 1) Compute OS : OL 8 on OCI for bastion server
### 2) MDS Shape : MySQL.HeatWave.VM.Standard.E3, enabled lakehouse 
### 3) MySQL version: 8.0.34

## 2. 실행 코드
### 1) sql 실행 명령어 (mysqlsh)
```
create database mydemo;
use mydemo;

SET @db_list = '["mydemo"]';
SET @par = 'https://idazzjlcjqzj.objectstorage.ap-seoul-1.oci.customer-oci.com/n/idazzjlcjqzj/b/iot-csv/o/iot_telemetry_data02.csv';
SET @ext_tables = '[{"db_name": "mydemo", "tables": [{"table_name": "iot_data", "dialect": {"format": "csv", "skip_rows": 1,  "date_format": "auto", "time_format": "auto","trim_spaces": true,
                                         '>  "field_delimiter": ",","record_delimiter": "\\n"}, "file": [{"par": "https://objectstorage.us-ashburn-1.oraclecloud.com/p/Pgz9ZJ91fRQs0Wlnzkr-eZnhSz7BGAsVeWwhEigna42PuWTOD7oQECjCvprJcAAs/n/idazzjlcjqzj/b/HWdatalake/o/airlines.csv" }] }] }]';

SET @options = JSON_OBJECT('mode', 'normal', 'external_tables', CAST(@ext_tables AS JSON));
call sys.heatwave_load(@db_list, @options);
```
### 2) 실행 캡쳐 화면
![image](https://github.com/mysqlsumi/hw-lakehouse/assets/31054795/1c8f263e-541b-4886-9706-964ccc2e0693)

![image](https://github.com/mysqlsumi/hw-lakehouse/assets/31054795/0b225571-be42-465f-a66a-38eaa46fb3a9)

## 3. 참고자료
https://blogs.oracle.com/mysql/post/getting-started-with-mysql-heatwave-lakehouse


