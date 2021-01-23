# COVID WORKSHOP

Name: Yesid Leonardo López Sierra  
Course: Cloud Computing  

# Procedimiento

The following workshop teach you how to create an ETL, connect it to databricks and create visualizations using Power BI.

## 1. Create the target bucket

First, let's create our S3 bucket in AWS with the following name: `covid-workshop`

Let the other configuration as default and click on then `create bucket` button.

![s3 config](https://github.com/leonleo997/covid-workshop/blob/master/assets/images/s3-config.PNG?raw=true)

## 2. ETL Process

Go to https://community.cloud.databricks.com/ 

### 2.1 Create a cluster
Go to section and click on `Create Cluster` with `covid-cluster` as the name.

![cluster config](https://github.com/leonleo997/covid-workshop/blob/master/assets/images/databricks-cluster.PNG?raw=true)

### 2.2 Create a notebook

Go to Workspace and create a new notebook with:

- name: covid-notebook
- Language: python
- Cluster: covid-cluster (the created cluster)

### 2.3 Add ETL process

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