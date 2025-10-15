```sql
CREATE OR REPLACE PROCEDURE <SCHEMA NAME>_<DIM/FACT><TABLE NAME>_ADD("TABLENAME" VARCHAR(16777216))
RETURNS STRING
LANGUAGE SQL
COMMENT='SP to add and update records to <SCHEMA NAME>.<TABLE NAME>'
EXECUTE AS OWNER
AS '
/*-------------------------------------------------------------------------------------------------
 This procedure is merging data from conformed table CONFORMED.<TABLE NAME>
 into <SCHEMA NAME>.<TABLE NAME>.
 Rules:
 <Explain logic>
 --------------------------------------------------------------------------------------------------
 Revision History
 --------------------------------------------------------------------------------------------------
 Date            Developer                  Comments
 --------------------------------------------------------------------------------------------------
 YYYY-MM-DD      <YOUR NAME>                Initial Revision
 YYYY-MM-DD      <YOUR NAME>                <Comments for new version>
 --------------------------------------------------------------------------------------------------
 */
DECLARE
    STRPROCEDURE              VARCHAR DEFAULT (SELECT PROCEDURE_NAME FROM CONFORMED.ETL_CONTROL_FLOW WHERE UPPER(TARGET_TABLE_NAME)=:TABLENAME); 
    STATUS                    VARCHAR DEFAULT ''SUCCEEDED'';
    SERRM                     VARCHAR DEFAULT ''No Errors'';
    SCODE                     NUMBER  DEFAULT 0;  
    SSTATE                    VARCHAR DEFAULT '''';
    SOURCE_TABLE              VARCHAR DEFAULT (SELECT SOURCE_SCHEMANAME||''.''||SOURCE_TABLE_NAME FROM CONFORMED.ETL_CONTROL_FLOW WHERE UPPER(TARGET_TABLE_NAME)=:TABLENAME);
    TARGET_TABLE              VARCHAR DEFAULT (SELECT TARGET_SCHEMANAME||''.''||TARGET_TABLE_NAME FROM CONFORMED.ETL_CONTROL_FLOW WHERE UPPER(TARGET_TABLE_NAME)=:TABLENAME);
    RECORD_COUNT              NUMBER DEFAULT 0; 

BEGIN
    MERGE INTO IDENTIFIER(:TARGET_TABLE) tgt -- <OR INSERT OR UPDATE etc. Can be multiple>
    USING IDENTIFIER(:SOURCE_TABLE) src
    <YOUR LOGIC>;

    -- Checking the record count in the target table  
        SELECT COUNT(*) INTO :RECORD_COUNT
        FROM IDENTIFIER(:TARGET_TABLE)
        WHERE CAST(CREATEDON AS DATE) = CURRENT_DATE();

    -- Check if the Record Count is Greater Than 0 and Send Success Email Notification  
        IF (:RECORD_COUNT > 0) THEN
        RETURN :STATUS||'' - ''||:SERRM;	
        ELSE
            LET SERRM := ''NO RECORDS LOADED'';
            RETURN :STATUS||'' - ''||:SERRM;
        END IF;

EXCEPTION
    WHEN OTHER THEN
    LET SCODE  := SQLCODE;
    LET SSTATE := SQLSTATE;
    LET SERRM  := REPLACE(SQLERRM,'''''''','''');
    LET STATUS := ''Failed'';                
    CALL ADMIN.ERROR_LOG_SQL_ADD(:STRPROCEDURE, :SCODE  , :SSTATE , :SERRM ,:STATUS, ''GTS'');      
    RAISE;  

END'
;
```
Example,
```sql
CREATE OR REPLACE PROCEDURE PBIGOV_DIMDATASETUPSTREAMDATASETS_ADD("TABLENAME" VARCHAR(16777216))
RETURNS STRING
LANGUAGE SQL
COMMENT='SP to add and update records to PBIGOV.DIMDATASETUPSTREAMDATASETS'
EXECUTE AS OWNER
AS '
/*-------------------------------------------------------------------------------------------------

 This procedure is merging data from conformed table CONFORMED.V_DIMPBIGOV_DATASETUPSTREAMDATASETS
 into PBIGOV.DIMDATASETUPSTREAMDATASETS.
 Rules:
 1. Rewrite new data
 --------------------------------------------------------------------------------------------------
 Revision History
 --------------------------------------------------------------------------------------------------
 Date            Developer                  Comments
 --------------------------------------------------------------------------------------------------
 2025-03-04      Ruslan Zolotukhin          Initial Revision
 --------------------------------------------------------------------------------------------------
 */
DECLARE
    STRPROCEDURE              VARCHAR DEFAULT (SELECT PROCEDURE_NAME FROM CONFORMED.ETL_CONTROL_FLOW WHERE UPPER(TARGET_TABLE_NAME)=:TABLENAME); 
    STATUS                    VARCHAR DEFAULT ''SUCCEEDED'';
    SERRM                     VARCHAR DEFAULT ''No Errors'';
    SCODE                     NUMBER  DEFAULT 0;  
    SSTATE                    VARCHAR DEFAULT '''';
    SOURCE_TABLE              VARCHAR DEFAULT (SELECT SOURCE_SCHEMANAME||''.''||SOURCE_TABLE_NAME FROM CONFORMED.ETL_CONTROL_FLOW WHERE UPPER(TARGET_TABLE_NAME)=:TABLENAME);
    TARGET_TABLE              VARCHAR DEFAULT (SELECT TARGET_SCHEMANAME||''.''||TARGET_TABLE_NAME FROM CONFORMED.ETL_CONTROL_FLOW WHERE UPPER(TARGET_TABLE_NAME)=:TABLENAME);
    RECORD_COUNT              NUMBER DEFAULT 0;  
BEGIN
    INSERT OVERWRITE INTO IDENTIFIER(:TARGET_TABLE)
       (
        DATASETID,
        TARGETDATASETID,
        GROUPID,
        EXTRACTEDON 
        )
    SELECT 
        DATASETID,
        TARGETDATASETID,
        GROUPID,
        EXTRACTEDON 
    FROM IDENTIFIER(:SOURCE_TABLE);

    -- Checking the record count in the target table  
        SELECT COUNT(*) INTO :RECORD_COUNT
        FROM IDENTIFIER(:TARGET_TABLE)
        WHERE CAST(CREATEDON AS DATE) = CURRENT_DATE();

    -- Check if the Record Count is Greater Than 0 and Send Success Email Notification  
        IF (:RECORD_COUNT > 0) THEN
        RETURN :STATUS||'' - ''||:SERRM;	
        ELSE
            LET SERRM := ''NO RECORDS LOADED'';
            RETURN :STATUS||'' - ''||:SERRM;
        END IF;

EXCEPTION
    WHEN OTHER THEN
    LET SCODE  := SQLCODE;
    LET SSTATE := SQLSTATE;
    LET SERRM  := REPLACE(SQLERRM,'''''''','''');
    LET STATUS := ''Failed'';                
    CALL ADMIN.ERROR_LOG_SQL_ADD(:STRPROCEDURE, :SCODE  , :SSTATE , :SERRM ,:STATUS, ''GTS'');      
    RAISE;  

END'
;

```
