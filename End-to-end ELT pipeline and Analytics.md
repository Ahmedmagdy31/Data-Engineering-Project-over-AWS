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


