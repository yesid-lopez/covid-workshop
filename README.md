# COVID WORKSHOP

Name: Yesid Leonardo López Sierra  
Course: Cloud Computing  

# Procedimiento

The following workshop teach you how to create an ETL, connect it to databricks and create visualizations using Power BI.

## 1. Create the target bucket

First, let's create our S3 bucket in AWS with the following name: `covid-workshop`

Let the other configuration as default and click on then `create bucket` button.

## 2. ETL Process

Go to https://community.cloud.databricks.com/ 

### 2.1 Create a cluster
Go to section and click on `Create Cluster` with `covid-cluster` as the name.

[databricks-cluster.png]

### 2.2 Create a notebook

Go to Workspace and create a new notebook with:

- name: covid-notebook
- Language: python
- Cluster: covid-cluster (the created cluster)

### 2.3 Add ETL process

In the created notebook add the following code to read the data source


```python
url = "https://storage.googleapis.com/covid19-open-data/v2/main.csv"
from pyspark import SparkFiles
spark.sparkContext.addFile(url)

df = spark.read.csv("file://"+SparkFiles.get("main.csv"), header=True, inferSchema= True)
```

then create a mount with the `covid-workshop` bucket. For that, you need two variables  `access_key` and `secret_key` (for that, you need to create a role in the IAM with full permissions for S3)