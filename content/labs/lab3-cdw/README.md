# Lab 3 - Cloudera Data Warehouse

In this lab you will take a tour of Cloudera Data Warehouse by querying data from the Open Data Lakehouse and building a dashboard to visualize the results.

**Goals**

- [ ] Configure Cloudera Data Warehouse
- [ ] Query data in Iceberg
- [ ] Created a visualization within Cloudera Data Vizualization

![Lab 3](lab3_cdw.png)

## Requirements

Ensure the following requirements are met in order to run this exercise:

- [ ] Name of the Impala Data Warehouse assigned to your user
- [ ] Name of Data Vizualization Cluster assigned to your user

!!! info
    The workshop instructor will assign your user to a Data Warehouse and Data Vizualisation cluster.

## Tasks

### Open Cloudera Data Warehouse

From the Cloudera on cloud Tenant home page, click on Data Warehouse.

![Cloudera on Cloud Tenant Home](cloudera_on_cloud_tenant_home.png)

In the pop up window that appears, click on **Manage Data Warehouse** to open the Data Warehouse Home page.

![Cloudera on Cloud Tenant Home](cloudera_on_cloud_tenant_home_cdw.png)

The Data Warehouse Home shows the deployed database catalogs, and virtual warehouses and allows access to data vizualization clusters.

Click on **Data Vizualization** from the left menu.

![Cloudera Data Warehouse](cdw_home_page.png)

In Data Visualization, click on the **Data Viz** button for the cluster instance your user was assigned.

![CDW Open Data Viz](cdw_open_data_viz.png)

### Create a Dataset in Data Vizualization

Once in Data Visualization, go to the **Data** option from the top menu, and then to the Connector of the form _viz-impala-cxn-00N_ from the left menu.

![CDW Data Viz Connectors](cdw_dv_connectors.png)

We have to create a new data source, for that, click on **New Dataset**. A window will
appear to enter the information of the new data source.

![CDW Data Viz New Dataset](cdw_dv_new_dataset.png)

Enter the information for the new data source as follows:

- **Dataset title**: Use the naming convention `<Workload Username>.telco_curated_data`
- **Dataset Source**: Select `From table`
- **Select Database**: Enter your Workload Username
- **Select Table**: Select `telco_data_curated`

Click on **CREATE** to create the new Dataset.

![CDW Data Viz Create New Dataset](cdw_dv_create_new_dataset.png)

### Explore Dataset in Data Vizualization

The new Dataset should appear in the list. Click on the dataset that you just created.

![CDW Data Viz Open New Dataset](cdw_dv_open_new_dataset.png)

Here you will see the details of the dataset.

![CDW Data Viz New Dataset Details](cdw_dv_new_dataset_details.png)

Click on **Fields** (left menu) to see the fields automatically captured during the dataset creation
process.

![CDW Data Viz Dataset Fields](cdw_dv_dataset_fields.png)

You can also preview the data from this screen. Click on **Data Model** (left menu) and then on the button **Show Data** that appears in the center.

![CDW Data Viz Dataset Preview Data](cdw_dv_dataset_preview_data.png)

At this moment, a query to the Virtual Warehouse is executed to retrieve the data from the data set. This may take 1-2 minutes to complete. Once loaded, notice the columns and values.

![CDW Data Viz Dataset Preview Data View](cdw_dv_dataset_preview_data_view.png)

### Create Data Vizualization Dashboard

Click **New Dashboard** to create a new dashboard.

![CDW Data Viz Create New Dashboard](cdw_dv_create_new_dashboard.png)

When opening the design canvas of a new panel, remove the element that is added by default, by clicking on the three dots (`...`) button at the top right of the element, and then clicking on the option **Delete Visual**

![CDW Data Viz New Dashboard](cdw_dv_new_dashboard.png)

At the top of the canvas, in the enter title field, enter the name `Churn Analysis` to identify the dashboard.

![CDW Data Viz Dashboard Step 1](cdw_dv_dashboard_step1.png)

To add a new visual element, click on the button **Visuals** from the right menu, select the dataset that corresponds to them, and click on the button **New Visual**.

![CDW Data Viz Dashboard Step 2](cdw_dv_dashboard_step2.png)

Add the first visual element, which is a Pie Chart with the Dimensions **churn** and **contract**, with the Measures of **Record count**. Once finished, click the button **Refresh Visual**. Set the title of the visual to `Churn by type of contract`.

![CDW Data Viz Dashboard Step 3](cdw_dv_dashboard_step3.png)

Add the second visual element, which is a Scatter Chart with the dimension **partner** as X Axis, **gender** as Y Axis, **dependents** as Colors and **avg (total charges)** as Size. Once finished, click the button **Refresh Visual**. Set the title of the visual to `Churn by family`.

![CDW Data Viz Dashboard Step 4](cdw_dv_dashboard_step4.png)

Add the third visual element, which is a Bar Chart with the dimensions **streamingtv** and **streamingmovies** as X Axis, **Record Count** as Y Axis and **churn** as Colors. Once finished, click the button **Refresh Visual**. Set the title of the visual to `Churn by service`.

![CDW Data Viz Dashboard Step 5](cdw_dv_dashboard_step5.png)

Add the fourth and last visual element, which is a table with the dimensions and metrics of the dataset. Be sure to add all 18 dimensions and 3 metrics (do not use Record Count) to the table. Once finished, click the button **Refresh Visual**. Set the title of the table to `Scoring - Churn Probability`.

![CDW Data Viz Dashboard Step 6](cdw_dv_dashboard_step6.png)

Save the dashboard by clicking the button **Save** from the top menu.

**End of Lab 3**