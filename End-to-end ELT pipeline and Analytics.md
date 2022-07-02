In this project I am building and executing end-to-end ELT pipeline and driving analytics using Amazon Redshift as the data warehouse solution. With the below AWS services

Amazon Redshift: Data warehouse and data lake

AWS Step Function: ELT orchestration

AWS Glue: Data catalog, ELT job and crawler

AWS Secrets Manager: Store Amazon Redshift cluster credential

Amazon Quicksight: BI tool

As a starting point I'm using NYC taxi sample data which is made available as gzip csv files in an Amazon S3 bucket. This data will be loaded, transformed, exported to data lake using Amazon Redshift. The exported data will be crawled via AWS Glue Crawler. This crawled table will be accessed via Amazon Redshift Spectrum. These steps are orchestrated using AWS Step function.
_________________________________________________________________________________________________________________________________________
1- LAUNCHING THE CLOUDFORMATION TEMPLATE
----------------------------------------
![template1-designer](https://user-images.githubusercontent.com/93045990/177006535-5b243d36-2ca3-444d-a1b1-0865c03e80d9.png)


![CFDiagram](https://user-images.githubusercontent.com/93045990/177006562-ec2b58a7-2725-4c7f-a53b-c780f5858d05.PNG)
2- SECRETS MANAGER
------------------
After the cloud formation deployment completing we check the secrets in the secrets managers such as (Redshift username, password)
![Secretsmanager](https://user-images.githubusercontent.com/93045990/177006772-d02cded1-6704-4fd8-a5e2-bfeafc708bb5.PNG)

__________________________________________________________________________________________________________________________________________
3- AMAZON REDSHIFT
------------------
We will do few steps here with redshift.
a. We will copy the ARN (Amazon Resource Name) under cluster permission related to IAM role.
b. From the editor in the left menu in editor: We will connect to the database using the credentials in the secrets manager mentioned before.
Database        :  nyctaxi
User    	:  master
Password	:  (from secrets manager console)
c. In the query editor we will create the schema and the main table containing the trips using the following SQL code (Please change the region and the ARN to yours)
![CREATINGSCHEMA-TABLE-REDSHIFT](https://user-images.githubusercontent.com/93045990/177007197-2ebb2f33-7ce0-46c5-97ca-1f969b03cf7f.PNG)

____________________________________________________________________________________________________________________________________________
AWS GLUE
--------
In Amazon AWS Glue we will check and edit the connection which has been already created in our Cloudformation template.
a. We will edit the connection and add our parameters of the database-name, user-name and password from the secrets manager as per the following screenshot.

![Glue-conn](https://user-images.githubusercontent.com/93045990/177007515-d3ac81bf-9105-441d-9735-a5995bb9dbd6.PNG)

b. From ETL > Job select the job name "rs-query" and then Action > Edit job (I worked on the legacy one):
we choose "rs-con".
![glue-edit-etl](https://user-images.githubusercontent.com/93045990/177007648-86031955-a5f6-47b2-b995-a6a3075850f0.PNG)

____________________________________________________________________________________________________________________________________________

AMAZON REDSHIFT 
---------------
*First we will do the data loading using the following script:
> SET query_group TO 'ingest';

>CREATE TEMPORARY TABLE nyc_greentaxi_tmp (LIKE taxischema.nyc_greentaxi);

>COPY nyc_greentaxi_tmp FROM '{0}' IAM_ROLE '{1}' csv ignoreheader 1 region '{2}' gzip;

>DELETE FROM taxischema.nyc_greentaxi USING nyc_greentaxi_tmp WHERE taxischema.nyc_greentaxi.vendorid = nyc_greentaxi_tmp.vendorid AND taxischema.nyc_greentaxi.lpep_pickup_datetime = nyc_greentaxi_tmp.lpep_pickup_datetime;

>INSERT INTO taxischema.nyc_greentaxi SELECT * FROM nyc_greentaxi_tmp;

>DROP TABLE nyc_greentaxi_tmp;

Don't forget to replace {0} by S3 bucket which has the data, {1} with ARN role, {2} with the region you are in.


*Second we will do the transormation and loading using the following script:

>SET query_group TO 'ingest';

>UNLOAD(
'select case when vendorid = 1 then ''Creative Mobile Technologies, LLC'' else ''VeriFone Inc.'' end as vendor
, date_part(hr, lpep_pickup_datetime)::int hour_pickup
, date_part(hr, lpep_dropoff_datetime)::int hour_dropoff
, trunc(lpep_pickup_datetime) pickup_date
, trunc(lpep_dropoff_datetime) dropoff_date
, case when RateCodeID = 1 then ''Standard rate''
       when RateCodeID = 2 then ''JFK''
       when RateCodeID = 3 then ''Newark''
       when RateCodeID = 4 then ''Nassau or Westchester''
       when RateCodeID = 5 then ''Negotiated fare''
       else ''Group ride''  end RateCode
, case when Payment_type = 1 then ''Credit car''
       when Payment_type = 2 then ''Cash''
       when Payment_type = 3 then ''No charge''
       when Payment_type = 4 then ''Dispute''
       when Payment_type = 5 then ''Unknown''
       else ''Voided trip''  end Paymenttype
, case when Trip_type = 1 then ''Street-hail''
       else ''Dispatch'' end Trip_type
, pulocationid
, dolocationid
, Passenger_count
, Trip_distance
, Total_amount
, Tip_amount
, Fare_amount
, tolls_amount
, total_amount
from taxischema.nyc_greentaxi
ORDER BY pickup_date'
)
to '{0}'
IAM_ROLE '{1}'
parquet
PARTITION BY (pickup_date)
ALLOWOVERWRITE
>;

___________________________________________________________________________________________________________________________________________
STEP FUNCTIONS
--------------
Step functions is used for orchesteration, we will find the job is already created so all we need is to start the execution.

![Stepfunctions2](https://user-images.githubusercontent.com/93045990/177009660-9cb9dbd3-4df7-45ac-8583-28e77a3cc04a.PNG)

____________________________________________________________________________________________________________________________________________

AWS GLUE
---------
We will use crawlers to crawl the data from the s3 bucket in parquet folder which we brought from the unload step, to crawl this data to a default database and create the table for us.

____________________________________________________________________________________________________________________________________________

AMAZON QUICKSIGHT
------------------

Using Quicksight we will create a dashboard with more than *3,000,000 records

![quicksightdb](https://user-images.githubusercontent.com/93045990/177011295-b9b49265-62f8-40e7-a433-9b0dc1ff3f7f.PNG)

![quicksightdb2](https://user-images.githubusercontent.com/93045990/177011301-7a2f762f-f4d0-41f3-935b-20e98c29a527.PNG)

![quicksightdb3](https://user-images.githubusercontent.com/93045990/177011728-8ae33d5d-ec7d-47a9-8d56-feb4440bfc09.PNG)
![quicksightdb4](https://user-images.githubusercontent.com/93045990/177011735-e1afc6e8-0f77-4d27-b142-b33130a84c7e.PNG)








