In this workshop I am building and executing end-to-end ELT pipeline and driving analytics using Amazon Redshift as the data warehouse solution. With the below AWS services

Amazon Redshift: Data warehouse and data lake

AWS Step Function: ELT orchestration

AWS Glue: Data catalog, ELT job and crawler

AWS Secrets Manager: Store Amazon Redshift cluster credential

Amazon Quicksight: BI tool

As a starting point I'm using NYC taxi sample data which is made available as gzip csv files in an Amazon S3 bucket. This data will be loaded, transformed, exported to data lake using Amazon Redshift. The exported data will be crawled via AWS Glue Crawler. This crawled table will be accessed via Amazon Redshift Spectrum. These steps are orchestrated using AWS Step function.

