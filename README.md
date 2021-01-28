# COVID WORKSHOP

Name: Yesid Leonardo López Sierra  
Course: Cloud Computing  



The following workshop teach you how to create an ETL, connect it to databricks and create visualizations using Power BI.

# 1. Create the target bucket

First, let's create our S3 bucket in AWS with the following name: `covid-workshop`

Let the other configuration as default and click on then `create bucket` button.

![s3 config](https://github.com/leonleo997/covid-workshop/blob/master/assets/images/s3-config.PNG?raw=true)

# 2. ETL Process

Go to https://community.cloud.databricks.com/ 

## 2.1 Create a cluster
Go to section and click on `Create Cluster` with `covid-cluster` as the name.

![cluster config](https://github.com/leonleo997/covid-workshop/blob/master/assets/images/databricks-cluster.PNG?raw=true)

## 2.2 Create a notebook

Go to Workspace and create a new notebook with:

- name: covid-notebook
- Language: python
- Cluster: covid-cluster (the created cluster)

## 2.3 Add ETL process

In the created notebook add the following code to read the data source.

```python
url = "https://storage.googleapis.com/covid19-open-data/v2/main.csv"
from pyspark import SparkFiles
spark.sparkContext.addFile(url)

df = spark.read.csv("file://"+SparkFiles.get("main.csv"), header=True, inferSchema= True)
```

then create a mount with the `covid-workshop` bucket. For that, you need two variables  `ACCESS_KEY` and `SECRET_KEY` (for that, you need to create a role in the IAM with full permissions for S3).

```python
access_key = "[ACCESS_KEY]"
secret_key = "[SECRET_KEY]"
encoded_secret_key = secret_key.replace("/", "%2F")
aws_bucket_name = "covid-workshop"
mount_name = "yelopezcovid"

dbutils.fs.mount(
  source = "s3a://%s:%s@%s" % (access_key, encoded_secret_key, aws_bucket_name), 
  mount_point = "/mnt/%s" % mount_name)
```
Finally, write the source dataset in the `covid-workshop` bucket (the target dataset).

```python
output = '/mnt/yelopezcovid/covid.csv'
df.write.csv(output)
```
When you check the target bucket there should be a folder with the following files:

![bucket etl process](https://github.com/leonleo997/covid-workshop/blob/master/assets/images/s3-output-etl.PNG?raw=true)

**Click here to check the [Notebook](https://databricks-prod-cloudfront.cloud.databricks.com/public/4027ec902e239c93eaaa8714f173bcfc/344449423735776/2785392546371234/8354352135207898/latest.html)**

# 3. Consult data from our data
Before to create a query we need to create a table to execute the queries against it. 


## 3.1 External table creation  

First, let's create the delta table:

```python
archivo = "/mnt/yelopezcovid/delta/yelopezcovid/"
df.write.format("delta").mode("overwrite").option("overwriteSchema","true").save(archivo)
```

and create the external table:

```python
tabla = "CREATE TABLE covid USING DELTA LOCATION '/mnt/yelopezcovid/delta/yelopezcovid/'"
spark.sql(tabla)
```

And check that the table is created with 

```SQL
%sql
show tables;
```

The output should something like this:  

![table-creation](https://github.com/leonleo997/covid-workshop/blob/master/assets/images/table-creation.PNG?raw=true)

## 3.2 Executing Queries

Once the delta table is created, let's execute the following queries:  

* ¿How many existing cases are around the world?
* ¿Which are the most affected countries?  
* Identify the most critical points in the United States
* Death Rate 

### ¿How many existing cases are around the world?
```sql
%sql
SELECT count(1) as CONTAGIOS_NIVEL_GLOBAL FROM covid
```

### ¿Which are the most affected countries?
```sql
%sql
SELECT country_name, count(1) as CASES 
FROM covid
GROUP BY country_name
ORDER BY CASES DESC
```

### Identify the most critical points in the United States

```sql
%sql
SELECT subregion1_name, count(1) as cases
FROM covid
WHERE country_code='US'
GROUP BY subregion1_name
ORDER BY cases DESC
```

### Death Rate  

```sql
%sql
SELECT country_name, 
(SUM(new_deceased)/ IF(SUM(new_confirmed)=0,1,SUM(new_confirmed))) as MORTALIDAD
FROM covid
WHERE aggregation_level = 0
GROUP BY country_name
ORDER BY MORTALIDAD DESC
```

**Click here to check the [Notebook](https://databricks-prod-cloudfront.cloud.databricks.com/public/4027ec902e239c93eaaa8714f173bcfc/344449423735776/2785392546371234/8354352135207898/latest.html)**

# 4.Visualization with Power BI

## 4.1 Connect Power BI with Databricks

First, let's the configuration to connect Power BI with Databricks. For that, go to `Get Data` and type `Spark` in the search bar.

![power-bi config](https://github.com/leonleo997/covid-workshop/blob/master/assets/images/power-bi-config0.PNG?raw=true)

Then, in the cluster configuration, go to configuration and click on the `JDBC/ODBC` tab.

From the JDBC URL:

```
jdbc:spark://community.cloud.databricks.com:443/default;transportMode=http;ssl=1;httpPath=sql/protocolv1/o/AAAAA/BBBBB-jays123
```

Create the server URL from the JDBC URL:

```
https://community.cloud.databricks.com:443sql/protocolv1/o/AAAAA/BBBBB-jays123
```
![power-bi config](https://github.com/leonleo997/covid-workshop/blob/master/assets/images/power-bi-config.PNG?raw=true)

Select `http` as protocol and click on `Accept`.

Then Power BI will ask you for the credentials:

![power-bi config](https://github.com/leonleo997/covid-workshop/blob/master/assets/images/power-bi-config2.PNG?raw=true)

Add them and click on `connect`. Finally import the covid table.

![power-bi config](https://github.com/leonleo997/covid-workshop/blob/master/assets/images/power-bi-config3.PNG?raw=true).

## 4.2 Charts

You can drag and drop the fields to create different charts as the following:

![chart](https://github.com/leonleo997/covid-workshop/blob/master/assets/images/power-bi-chart.PNG?raw=true).

If you want to check better the chart click on this [link](https://github.com/leonleo997/covid-workshop/blob/master/assets/images/power-bi-chart.pdf)

