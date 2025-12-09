# trinotosnowflakeicebergrestinterface
Trino - Snowflake - Apache Iceberg - Horizon - Iceberg REST API (IRC) - Polaris API


Follow this
https://jasonhhughes.medium.com/trino-reading-apache-iceberg-tables-via-snowflakes-horizon-iceberg-rest-catalog-apis-a43bb8326466


From Snowflake


````

---------- Trino to Snowflake


CREATE USER TRINO_USER PASSWORD=NULL TYPE=SERVICE;
CREATE ROLE TRINO_ROLE;
GRANT ROLE TRINO_ROLE TO USER TRINO_USER;

SHOW AUTHENTICATION POLICIES;

ALTER AUTHENTICATION POLICY my_auth_policy
SET AUTHENTICATION_METHODS = ('OAUTH', 'PASSWORD', 'PROGRAMMATIC_ACCESS_TOKEN');

CREATE NETWORK RULE allow_public_access
MODE = INGRESS
TYPE = IPV4
VALUE_LIST = ('0.0.0.0/0');

CREATE NETWORK POLICY allow_public_policy
ALLOWED_NETWORK_RULE_LIST = ('allow_public_access');
  
ALTER USER TRINO_USER SET AUTHENTICATION POLICY my_auth_policy;
ALTER USER TRINO_USER SET NETWORK_POLICY = allow_public_policy;
  
ALTER USER TRINO_USER ADD PROGRAMMATIC ACCESS TOKEN iceberg_token 
ROLE_RESTRICTION = 'TRINO_ROLE';

SELECT 'https://'||REPLACE(CONCAT_WS('-', CURRENT_ORGANIZATION_NAME(), 
CURRENT_ACCOUNT_NAME()),'_','-')||'.snowflakecomputing.com/polaris/api/catalog' 
AS HIRC_URL;


GRANT MODIFY PROGRAMMATIC AUTHENTICATION METHODS ON USER TRINO_USER
  TO ROLE TRINO_ROLE;

SHOW USER PROGRAMMATIC ACCESS TOKENS FOR USER trino_user;


CREATE OR REPLACE AUTHENTICATION POLICY my_auth_policy
  PAT_POLICY=(
    NETWORK_POLICY_EVALUATION = ENFORCED_NOT_REQUIRED
  ),  AUTHENTICATION_METHODS = ('OAUTH', 'PASSWORD', 'PROGRAMMATIC_ACCESS_TOKEN');;        
        
GRANT USAGE ON DATABASE DEMO to ROLE TRINO_ROLE;
GRANT USAGE ON SCHEMA DEMO.DEMO TO ROLE TRINO_ROLE;
GRANT ALL ON TABLE DEMO.DEMO.ICYMTA 
TO ROLE TRINO_ROLE;

ALTER DATABASE DEMO SET 
EXTERNAL_VOLUME = TRANSCOM_TSPANNICEBERG_EXTVOL;

````


Note:  use lowercase for name, don't use underscores (_)

Note:   rest-catalog warehouse is database name with Permissions for the TRINO_ROLE

`````


CREATE CATALOG horizonsnow USING iceberg
WITH (
 "iceberg.rest-catalog.uri" = 'https://lowercase-yoursnowflake-aws1.snowflakecomputing.com/polaris/api/catalog',
 "iceberg.rest-catalog.oauth2.credential" = 'create a Programmatic access token tied to this role below',
 "iceberg.rest-catalog.oauth2.scope" = 'session:role:TRINO_ROLE',
 "s3.region" = 'us-east-1',
 "iceberg.rest-catalog.warehouse" = 'DEMO',
 "iceberg.catalog.type" = 'rest',
 "iceberg.rest-catalog.security" = 'OAUTH2',
 "iceberg.rest-catalog.case-insensitive-name-matching" = 'true',
 "iceberg.rest-catalog.view-endpoints-enabled" = 'false',
 "fs.native-s3.enabled" = 'true',
 "iceberg.rest-catalog.vended-credentials-enabled" = 'true'
);

CREATE CATALOG

trino> use horizonsnow.DEMO;
USE
trino:demo> show tables;
        Table        
---------------------
 flight_data_iceberg 
 icymta              
 planes              
 stockvalues         
(4 rows)

Query 20251209_011321_00005_q7bzc, FINISHED, 1 node
Splits: 11 total, 11 done (100.00%)
3.47 [4 rows, 310B] [1 rows/s, 89B/s]


trino:demo> select * from planes limit 20;
 seen_pos |  flight  |    adsbnow     | latitude  | nic | emergency |                 uuid                 | seen | geomrate | navheading |          navigationmodes    >
----------+----------+----------------+-----------+-----+-----------+--------------------------------------+------+----------+------------+----------------------------->
 20.9     | DAL849   | 1.7472308775E9 | 40.561011 | 8   | none      | afb7250d-3225-49d9-80ca-9d21775ad519 | 19.0 |          | 0.0        |                             >
 11.4     | AMX004   | 1.7472308775E9 | 40.754974 | 8   | none      | 4b2d5743-1e51-45fa-a62a-55df482d881e | 0.2  | 0        | 243.3      | ["autopilot","vnav","lnav",">
 4.1      | JBU2695  | 1.7472308775E9 | 40.282974 | 8   | none      | 501087c5-aa55-4345-bdfb-fe54e3aa0990 | 0.6  |          | 0.0        |                             >
 0.2      | DAL461   | 1.7471451092E9 | 40.320709 | 8   | none      | a0ed15d1-ae7a-4668-967d-91559b21ec59 | 0.0  | -1600    | 80.2       |                             >
 42.1     | RPA4511  | 1.7471450905E9 | 40.33461  | 8   | none      | f17ce872-6ff1-4ad8-ac0f-c1789c7c7367 | 11.2 |          |            | ["autopilot","althold","tcas>
 31.1     | EDV5015  | 1747145076     | 40.173019 | 8   |           | bba09841-155e-43dc-8e2b-5d5e1604f464 | 0.2  | 1600     | 275.6      |                             >
 27.6     | RPA4511  | 1747145076     | 40.33461  | 8   | none      | 7ed0e2c9-e674-41ce-9b17-f1f67ba2c9d0 | 1.6  |          |            | ["autopilot","althold","tcas>
 34.4     |          | 1.7471450449E9 | 40.458596 | 8   |           | 010b9717-1699-429b-a38c-1e575f4f9893 | 33.1 |          |            |                             >
 0.7      | RPA4511  | 1.7471450303E9 | 40.321174 | 8   | none      | ad02c17b-8435-4689-b2e6-e2477aea5456 | 0.0  |          |            | ["autopilot","tcas"]        >
 20.8     |          | 1.7471447761E9 | 40.450729 | 8   | none      | 3a8d595e-6fe7-4641-be95-84f320d821d7 | 5.1  |          | 0.0        |                             >
 54.8     | RPA4745  | 1.7471661393E9 | 40.391281 | 8   | none      | 411be51a-51c9-4a8e-8caa-28ffc73a9b1c | 51.5 |          |            | ["autopilot","tcas"]        >
 3.2      | JBU1555  | 1.7434319867E9 | 40.437057 | 8   | none      | 371e7a2f-22cc-4dbd-b402-d4e6f6c4c48f | 1.7  |          |            | ["autopilot","vnav","tcas"] >
 7.7      | JBU1555  | 1.7434320178E9 | 40.404177 | 8   |           | 9ccbd63e-dd97-4795-b2a7-95e596adb61d | 4.9  |          |            | ["autopilot","vnav","tcas"] >
 1.1      | RPA4581  | 1.7434320178E9 | 40.441651 | 8   |           | 9dc1041b-723c-4488-b272-60ec678d228f | 1.0  |          |            | ["autopilot","althold","tcas>
 38.8     | EJA573   | 1.7434320022E9 | 40.310852 | 8   |           | 39f3f113-02d4-436b-a4e9-75016070ebc3 | 19.4 |          |            |                             >
 64.7     |          | 1.7434318798E9 | 40.191879 | 8   |           | 3670c2a4-8f5a-4269-b7f5-a34be02368c7 | 5.2  |          |            |                             >
 0.6      | UAL2404  | 1.7434318798E9 | 40.26047  | 8   |           | db960f60-819f-423b-be5f-ced5ce23332d | 0.4  |          | 201.1      |                             >
 20.6     |          | 1.7434318642E9 | 40.535843 | 8   | none      | 528b7d80-09ef-42bc-9ae6-bcae5e9b3851 | 20.6 | 0        | 58.4       |                             >
 66.2     |          | 1.7434319099E9 | 40.535843 | 8   |           | 5c12cd9d-9943-4de1-8d0f-4a04b4d33b68 | 0.2  |          |            |                             >
 1.4      | RPA4377  | 1.7434319099E9 | 40.2891   | 8   | none      | 79a5ba13-5e32-42b7-9e56-8e31e1a87bf7 | 1.0  |          |            | ["autopilot","tcas"]        >
(20 rows)

Query 20251209_011711_00007_q7bzc, FINISHED, 1 node
http://localhost:8080/ui/query.html?20251209_011711_00007_q7bzc
Splits: 12 total, 12 done (100.00%)
CPU Time: 0.1s total,   272 rows/s, 319KiB/s, 23% active
Per Node: 0.0 parallelism,     3 rows/s, 4.33KiB/s
Parallelism: 0.0
Peak Memory: 20.5KiB
9.22 [34 rows, 39.9KiB] [3 rows/s, 4.33KiB/s]

trino:demo> select * from stockvalues limit 20;
                 uuid                 | lastprice | stockvolume | symbol |      ts       | tradeconditions 
--------------------------------------+-----------+-------------+--------+---------------+-----------------
 3ba6ab06-77d6-4999-b323-c09be8e0af0b |    131.98 |       200.0 | ABT    | 1743177174651 | [1, 8]          
 e8ec2443-871e-4db8-8d51-3a76c9701fa8 |  167.7777 |        51.0 | AVGO   | 1743177174651 | [1, 32, 12]     
 11d7dc98-6f4b-478d-9939-0f769fa0232a |   109.735 |        30.0 | NVDA   | 1743177174649 | [1, 12]         
 ac18430b-05d5-4435-9d49-ab20c938495f |    109.73 |        13.0 | NVDA   | 1743177173954 | [12]            
 507256db-a2b9-44da-9337-ba2ff710e079 |    59.017 |       100.0 | BMY    | 1743177174426 | [1, 32, 3]      
 c51c88c2-2d50-4497-ac31-228e6b28cb53 |    156.39 |        10.0 | GOOGL  | 1743177173766 | [1, 12]         
 6b79f494-4440-418b-aefd-fa15c0b7792b |    219.37 |         5.0 | AAPL   | 1743177173702 | [1, 12]         
 b89186ac-8c50-4aff-b370-dcd9ba614712 |    194.11 |       100.0 | AMZN   | 1743177173399 | [1, 8]          
 e8465484-7608-4011-8469-832b02972630 |    109.74 |         1.0 | NVDA   | 1743177172814 | [12]            
 861c251d-84a9-44e0-bd91-639b0570255c |    219.37 |        20.0 | AAPL   | 1743177172519 | [1, 12]         
 5acb51b1-cd70-410b-bb6a-037829b87c4b |  132.0071 |         1.0 | ABT    | 1743177173645 | [1, 12]         
 f90bc311-05e5-4082-ba5b-0fb60d01a2e2 |    156.39 |       100.0 | GOOGL  | 1743177173440 | [1]             
 09359840-6ce1-4c3f-8847-81b222ffff6e |    194.09 |         2.0 | AMZN   | 1743177172598 | [1, 12]         
 8044bfe1-0680-4ab5-93cf-18ce2cfe1a3f |    265.73 |        10.0 | TMUS   | 1743177172240 | [1, 8, 12]      
 d91e7f9c-eb9f-44fd-8dba-3da46c80a4c7 |   85.3401 |         5.0 | WMT    | 1743177171186 | [1, 12]         
 5d46ae2d-1235-416e-a762-9933f0a55f06 |    109.73 |        13.0 | NVDA   | 1743177174446 | [1, 12]         
 e7fa6677-efc1-4ed7-ae44-2350b1bbcb2f |  156.3882 |         1.0 | GOOGL  | 1743177173847 | [1, 12]         
 a2b647a6-746d-4fb8-ba4f-74860102d880 |   70.4515 |         4.0 | C      | 1743177173262 | [1, 12]         
 39ac9529-902b-4dfe-a421-0f277090b60e |  935.2912 |         1.0 | COST   | 1743177174058 | [1, 12]         
 02b03880-7c1c-429d-8df7-8895ccd4715d |  150.8258 |        20.0 | PEP    | 1743177173797 | [1, 32, 12]     
(20 rows)

Query 20251209_012209_00010_q7bzc, FINISHED, 1 node
Splits: 10 total, 10 done (100.00%)
8.14 [31 rows, 13MiB] [3 rows/s, 1.59MiB/s]


````
