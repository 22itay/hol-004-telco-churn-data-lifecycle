# Lab 2 - Cloudera Data Engineering

In this lab you will get an overview of Cloudera Data Engineering by configuring and deploying a Spark job to prepare the data.

**Goals**

- [ ] Run a data enrichment process
- [ ] Run a process to simulate changes to the data
- [ ] Configure the execution of a pipeline using low-code/no-code tools

![Lab 2](lab2_cde.png)

## Requirements

None.

## Tasks

### Open Cloudera Data Engineering

From the Cloudera on cloud Tenant home page, click on Data Engineering.

![Cloudera on Cloud Tenant Home](cloudera_on_cloud_tenant_home.png)

The Data Engineering Home shows all the actions that can be done, such as Jobs in Spark and pipelines in Airflow, Resources and useful information/documentation.

Click on **Jobs** from the left menu to create an Airflow job.

![Cloudera Data Engineering](cde_home_page.png)

### Create Airflow Jobs

!!! info
    For this workshop, two jobs have been created. We will create an Airflow job which uses both of these.

    * **CDE-updateTable** - generate random changes in rich table to visualize Lakehouse Time Travel functionality.
    * **CDE-telco_data_enrichment** - process in Spark (Python) to enrich the data ingested from Kafka and save to a new table.

In the Job window, ensure that the `{{ cde_virtual_cluster_name }}` Virtual Cluster is selected at the top of the windown. Then click on **Create Job**.

![Cloudera Data Engineering Create Job](cde_create_job.png)

In the Job creation form, enter the following information:

- **Job Type**: Airflow
- **Name**: Use the naming convention `USERNAME-pipeline`.
- **DAG**: Editor, to graphically configure the task.

Click **Create** once these values are correctly entered.

![Cloudera Data Engineering Save Job](cde_save_job.png)

On the Job editing screen, select the **Editor** tab, and you will see the canvas shown in the image below. This can be used to construct the steps of the pipeline. In our case, we are going to create two Cloudera Data Engineering Jobs and relate them.

![Cloudera Data Job Editor Canvas](cde_job_canvas.png)

### Configure Data Enrichment job

Let's start with the first Job - Data Enrichment.

Click on the CDE Job button and drag onto the canvas, entering the following settings:

- **Title/Name**: `Data Enrichment`
- **Select Job**: Select `CDE-telco_data_enrichment`
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

- Check the checkbox **Override Spark values**. Additional options will appear below.
- **Arguments**: Enter your Workload Username, `USERNAME`.

![Add Data Enrichment Job](cde_add_enrichment_job.png)

- Click **Save** button.

### Configure Table Update job

Configure the second Job - Table Update.

Click on the CDE Job button and drag onto the canvas, entering
the following settings:

- **Title/Name**: `Table Update`
- **Select Job**: Select `CDE-updateTable`
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
- Check the checkbox **Override Spark values**. Additional options will appear below.
- **Arguments**: Enter your Workload Username, `USERNAME`.

![Add Table Update Job](cde_add_table_update_job.png)

- Click **Save** button.

### Configure Job Execution Squenence

To set up the execution sequence, bind **Data Enrichment** with **Table Update**. For that, click on the right connector of the job of Data Enrichment and drag to the left connector of Table
Update.

![Job Sequence Step 1](cde_job_squence_step1.png)

![Job Sequence Step 2](cde_job_squence_step2.png)

![Job Sequence Step 3](cde_job_squence_step3.png)

Once the Jobs have been joined, click on **Save** to save the settings made. You should see a message indicating `Pipeline saved to job`.

![Save Job Sequence](cde_save_job_sequence.png)

### Run the Airflow Job

We can now run the pipeline.

On the upper right side of the canvas, click **Actions** -> **Run Now**.

![Run Job](cde_run_job.png)

You should see the pipeline execution screen, indicating that the execution has been initialized.

![Run Job Initialized](cde_run_job_initialized.png)

Click on the **Airflow UI** tab to see the execution detail of each step in the pipeline.

The configured Data Enrichment and Table Update jobs are listed at the bottom left. The colors indicating the status of each job. Make sure the radio button **Auto-refresh** is enabled to automatically display the status of jobs.

![Inspect Running Job](cde_run_job_inspect.png)

You can see more information about the execution by clicking on the **Graph** view. Hovering the mouse over the Job name displays specific information for each step in the pipeline.

![Inspect Running Job](cde_run_job_information.png)

Once both jobs go to status success, make sure the overall pipeline status is success by clicking on the **Details** view. If _Total success_ is 1 the execution was successful.

![Successful Job Execution](cde_run_job_success.png)

**End of Lab 2**