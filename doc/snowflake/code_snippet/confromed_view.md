```sql
CREATE OR REPLACE VIEW V_<VIEW NAME> (
        <COLUMN 1 NAME>, --view columns list 
        <COLUMN 2 NAME>,
        ...
        <COLUMN N NAME>,
        EXTRACTEDON, 
        HASH_VALUE
) AS
/*----------------------------------------------------------------------------------------------
 This view is for loading data into <SCHEMA NAME>.<TABLE NAME>
 -----------------------------------------------------------------------------------------------
 Revision History
 -----------------------------------------------------------------------------------------------
 Date            Developer                  Comments
 -----------------------------------------------------------------------------------------------
 YYYY-MM-DD      <YOUR NAME>                Initial Revision
 YYYY-MM-DD      <YOUR NAME>                <Comments for new version>
 -----------------------------------------------------------------------------------------------
 */
SELECT
    <COLUMN 1 NAME>,
    <COLUMN 2 NAME>,
    ...
    <COLUMN N NAME>,
    EXTRACTEDON, 
    HASH(<All Columns List except FRMW_EXTRACTED_ON>) AS HASH_VALUE 
FROM
    CONFORMED.<TABLE NAME>;
```
Example,
```sql
CREATE OR REPLACE VIEW V_PBIGOV_DIMDATASETUPSTREAMDATASETS(
	DATASETID,
	TARGETDATASETID,
	GROUPID,
        EXTRACTEDON, 
	HASH_VALUE
) AS
/*----------------------------------------------------------------------------------------------
 This view is for loading data into PBIGOV.DIMDATASETUPSTREAMDATASETS
 -----------------------------------------------------------------------------------------------
 Revision History
 -----------------------------------------------------------------------------------------------
 Date            Developer					Comments
 -----------------------------------------------------------------------------------------------
 2025-03-04      Ruslan Zolotukhin     		Initial Revision
 -----------------------------------------------------------------------------------------------
 */
SELECT
    DATASETID,
    TARGETDATASETID,
    GROUPID,
    FRMW_EXTRACTED_ON AS EXTRACTEDON, 
    HASH(DATASETID,TARGETDATASETID,GROUPID) AS HASH_VALUE
FROM 
    CONFORMED.PBIGOV_DATASETUPSTREAMDATASETS;

```

