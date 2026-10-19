# Lab 2 - Cloudera Data Engineering using AI workbench

In this lab you will get an overview of Cloudera Data Engineering by configuring and deploying a Spark job to prepare the data.

**Goals**

- [ ] Run a data enrichment process
- [ ] Run a process to simulate changes to the data

![Lab 2](lab2_cde.png)

## Requirements

None.

## Tasks

### Open Cloudera AI

From the Cloudera on cloud Tenant home page, click on Cloudera AI.

Enter into our AI Workbench - spr-hol-default-ml



!!! info
    For this workshop, two jobs have been created. We will run them in a jupyter notebook

    * **updateTable.py** - generate random changes in rich table to visualize Lakehouse Time Travel functionality.
    * **telco_data_enrichment.py** - process in Spark (Python) to enrich the data ingested from Kafka and save to a new table.



After entering the workbench click on create new project named user_<xxx>.telco_enrichment


### Run Data Enrichment job

Create a session - click on top right +new session
Choose editor - JupyterLab and choose enable spark

- **Title/Name**: `Data Enrichment`

```python
import cml.data_v1 as cmldata


CONNECTION_NAME = "spr-hol-aw-dl"
conn = cmldata.get_connection(CONNECTION_NAME)
spark = conn.get_spark_session()
username='<xxx>'
# Sample usage to run query through spark
EXAMPLE_SQL_QUERY = "show databases"
spark.sql(EXAMPLE_SQL_QUERY).show()


spark.sql("TRUNCATE TABLE spark_catalog.{}.telco_data_curated".format(username))

temp_df = spark.sql("SELECT multiplelines, paperlessbilling, d.description as gender, onlinesecurity, internetservice, techsupport, b.description as contract, churn, seniorcitizen, deviceprotection, streamingtv, streamingmovies, FLOAT(totalcharges), c.description as partner, FLOAT(monthlycharges), customerid, e.description as dependents, onlinebackup, phoneservice, FLOAT(tenure), paymentmethod FROM {}.telco_iceberg_kafka a JOIN master_data.contract b ON b.id = contract JOIN master_data.misc c ON c.id = partner JOIN master_data.misc d ON d.id = gender JOIN master_data.misc e ON e.id = dependents".format(username))
print (temp_df.show())
temp_df.writeTo("spark_catalog.{}.telco_data_curated".format(username)).append()
```

- **Arguments**: Enter your Workload Username, `USERNAME`.



- Click **Run** button.

### Configure Table Update job


- **Title/Name**: `Table Update`
```python
import os
from datetime import datetime
import sys
import configparser
from pyspark.sql import SparkSession
from pyspark.sql.types import *
import cml.data_v1 as cmldata


CONNECTION_NAME = "spr-hol-aw-dl"
conn = cmldata.get_connection(CONNECTION_NAME)
spark = conn.get_spark_session()

# Sample usage to run query through spark
EXAMPLE_SQL_QUERY = "show databases"
spark.sql(EXAMPLE_SQL_QUERY).show()

DATABASE='<xxx>'
print("Running job as Username: ", DATABASE)

#CREATE EXTERNAL TABLE ext_t1 STORED AS ICEBERG TBLPROPERTIES ('iceberg.table_identifier'='default.telco_iceberg');

from pyspark.sql.functions import rand,abs,when,regexp_replace
df=spark.read.format("iceberg").load("spark_catalog."+ DATABASE + ".telco_data_curated")

df=df.select('customerid','monthlycharges', 'tenure','churn').where(df.churn != 'Yes')
df=df.drop_duplicates()
df=df.drop_duplicates(['customerid'])
df=df.withColumn("monthlycharges",df.monthlycharges.cast(DoubleType()))
df=df.withColumn("tenure",df.tenure.cast(DoubleType()))

df=df.withColumn("monthlycharges",df["monthlycharges"]+df["monthlycharges"]*(2*rand()-1))
df=df.withColumn("monthlycharges",df.monthlycharges)

spark.sql("drop table if exists " + DATABASE + "modtable")
df.write.mode("overwrite").saveAsTable(DATABASE+"modtable")

spark.sql("MERGE INTO spark_catalog."+ DATABASE + ".telco_data_curated t USING "+DATABASE+"modtable s ON t.customerid = s.customerid WHEN MATCHED THEN UPDATE SET monthlycharges = s.monthlycharges")

spark.sql("drop table if exists " + DATABASE + "modtable")

```

- **Arguments**: Enter your Workload DATABASE, `USERNAME`.


- Click **Run** button.


**End of Lab 2**
