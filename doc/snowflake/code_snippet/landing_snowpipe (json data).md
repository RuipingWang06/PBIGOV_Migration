```sql
CREATE OR REPLACE PIPE PIPE_<TABLE NAME>
    AUTO_INGEST=TRUE
    AWS_SNS_TOPIC='<SNS TOPIC>'  
    ERROR_INTEGRATION = Pipe_Error_Notification_Send
    AS COPY INTO  
        LANDING.<TABLE NAME>
        (
            FRMW_EXTRACTED_ON,
            <COLUMN 1 NAME>, -- section to list fields of target LANDING table 
            <COLUMN 2 NAME>,
            ...
            <COLUMN N NAME>
        )
    FROM (
        SELECT
            METADATA$FILE_LAST_MODIFIED AS FRMW_EXTRACTED_ON,
            $1:<COLUMN 1 NAME>::STRING, -- section to list fields of source file 
            $1:<COLUMN 2 NAME>::STRING,
            ...
            $1:<COLUMN N NAME>::STRING
        FROM @STAGE_S3_EIP/data/<ROOT FOLDER NAME>/<FOLDER NAME>/ (FILE_FORMAT => 'LANDING.JSON_S3')
    )
    FILE_FORMAT = (type = 'JSON');
```
Assuming data source is JSON flat file in S3. 
Don't do any transformations in pipe, do them in CONFORMED view instead. 

SNS TOPIC you use: 

|Environment|SNS Topic| 
|--|--|
|SF DEV|arn:aws:sns:us-east-1:956501017073:sns-eip-dev-uea1-s3eb-object_created-001-topic|
|Repository/branch/SF STG/PRD|arn:aws:sns:us-east-1:731598133995:sns-eip-prd-uea1-s3eb-object_created-001-topic|

**Column names are case sensitive and should match exactly to names in S3. Otherwise you your columns will simply return blanks.**

Example,
```sql
CREATE OR REPLACE PIPE PIPE_PBIGOV_DATASETUPSTREAMDATASETS 
    AUTO_INGEST=TRUE
    AWS_SNS_TOPIC='arn:aws:sns:us-east-1:956501017073:sns-eip-dev-uea1-s3eb-object_created-001-topic'  
    ERROR_INTEGRATION = Pipe_Error_Notification_Send
    AS COPY INTO   
        LANDING.PBIGOV_DATASETUPSTREAMDATASETS
        (
            FRMW_EXTRACTED_ON,
            DATASETID,
            TARGETDATASETID,
            GROUPID
        )
    FROM (
            SELECT
                METADATA$FILE_LAST_MODIFIED AS FRMW_EXTRACTED_ON,
                $1:DATASETID::STRING,
                $1:TARGETDATASETID::STRING,
                $1:GROUPID::STRING
            FROM @STAGE_S3_EIP/data/pbi/datasetupstreamdatasets_scan/ (FILE_FORMAT => 'LANDING.JSON_S3') 
    ) 
    FILE_FORMAT = (type = 'JSON');
```

For debugging you can run query to see all available data:
```sql
        SELECT TOP 1 
            $1::STRING
        FROM @STAGE_S3_EIP/data/<ROOT FOLDER NAME>/<FOLDER NAME>/ (FILE_FORMAT => 'LANDING.JSON_S3')
```
**Note. It could fail if you have a lot of data.** 

To activate Snowpipe use command:
```sql
ALTER PIPE <Pipe Name> SET PIPE_EXECUTION_PAUSED = FALSE;
```

To deactivate Snowpipe use command:
```sql
ALTER PIPE <Pipe Name> SET PIPE_EXECUTION_PAUSED = TRUE;
```

Note: If files were dropped before we created the pipe with the correct configuration, you will need to do a manual load by executing the COPY INTO command of the pipe directly in the database.

Example,
```sql
COPY INTO  
        LANDING.PBIGOV_DATASETUPSTREAMDATASETS
    FROM (
        SELECT
            $1:DatasetId::STRING,
            $1:TargetDatasetId::STRING,
            $1:GroupId::STRING
        FROM @STAGE_S3_EIP/data/pbi/datasetupstreamdatasets/ (FILE_FORMAT => 'LANDING.JSON_S3')
    )
    FILE_FORMAT = (type = 'JSON');
```
