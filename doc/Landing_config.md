You need to run the script
```sql
INSERT INTO DEV.LANDING.TABLEMETADATA (
    TABLENAME, 
    COLUMNKEYS, 
    SORTCOLUMN)
VALUES (
    '<TABLE NAME>', 
    '<KEY COLUMN LIST>', 
    '<SORT BY COLUMN>');
```
All fields are required to populate. Note Table name without schema name.

**Note. It is needed only if we need to remove duplicates when loading from LANDING to CONFORMED.**

Example,
```sql
INSERT INTO DEV.LANDING.TABLEMETADATA (
    TABLENAME, 
    COLUMNKEYS, 
    SORTCOLUMN)
VALUES (
    'PBIGOV_DATASETUPSTREAMDATASETS', 
    'DATASETID,TARGETDATASETID', 
    'DATASETID');
```
