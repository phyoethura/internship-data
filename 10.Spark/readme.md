# Spark Cluster Deployment with Helm and Running a PySpark Job

This guide outlines the steps to deploy an Apache Spark cluster using Helm, download a sample CSV file, run a PySpark job on the cluster, and analyze data using PySpark.

## Prerequisites

- Kubernetes cluster
- Helm installed
- kubectl installed
- Docker registry access (for pulling images)

## Steps

### 1. Add Bitnami Helm Repository

First, add the Bitnami Helm repository to your Helm configuration.

```sh
helm repo add bitnami https://charts.bitnami.com/bitnami
```

### 2. Search for Available Charts

Search the repository to see available charts.

```sh
helm search repo bitnami
```

### 3. Install Spark using Helm

Install the Spark Helm chart from Bitnami.

```sh
helm install wky-spark oci://registry-1.docker.io/bitnamicharts/spark --create-namespace spark
```

### 4. Upgrade Spark Helm Deployment

If you need to upgrade the Spark deployment with specific values, you can use the following command.

```sh
helm upgrade wky-spark bitnami/spark --values values.yaml --version 7.1.0 --namespace spark
```

### 5. Download Sample Data

Download the sample CSV file that will be used for the PySpark job.

```sh
curl -L "https://drive.google.com/uc?id=1VEi-dnEh4RbBKa97fyl_Eenkvu2NC6ki&export=download" -o people.csv
```

### 6. PySpark Script

Create a PySpark script `readcsv.py` to read the CSV file and perform some simple operations.

```python name=readcsv.py
from pyspark.sql import SparkSession
#from pyspark.sql.functions import sum
from pyspark.context import SparkContext

spark = SparkSession\
 .builder\
 .appName("Mahla")\
 .getOrCreate()


sc = spark.sparkContext

path = "people.csv"

df = spark.read.options(delimiter=",", header=True).csv(path)

df.show()

#df.groupBy("Job Title").sum().show()

df.createOrReplaceTempView("Peopletable")
df2 = spark.sql("select Sex, count(1) countsex, sum(Index) sex_sum " \
                     "from peopletable group by Sex")

df2.show()

#df.select(sum(df.Index)).show()
```

### 7. Copy Files to Spark Master

Copy the CSV file and the PySpark script to the Spark master pod.

```sh
kubectl cp people.csv wky-spark-master-0:/opt/bitnami/spark/examples/src/main/resources -n spark
kubectl cp readcsv.py wky-spark-master-0:/opt/bitnami/spark -n spark
```

### 8. Execute PySpark Job

Open a bash session in the Spark master pod and run the PySpark job.

```sh
kubectl exec -it wky-spark-master-0 -- /bin/bash
```

Inside the pod, run:

```sh
./bin/spark-submit \
 --class org.apache.spark.examples.SparkPi \
 --master spark://wky-spark-master-0.wky-spark-headless.spark.svc.cluster.local:7077 \
 /opt/bitnami/spark/examples/jars/spark-examples_2.12-3.4.1.jar 1000
```

To run the custom PySpark script:

```sh
./bin/spark-submit \
 --class org.apache.spark.examples.SparkPi \
 --master spark://wky-spark-master-0.wky-spark-headless.spark.svc.cluster.local:7077 \
 /opt/bitnami/spark/readcsv.py
```

### Notes

- Ensure your Kubernetes cluster has enough resources to handle the Spark deployment.
- Modify the `values.yaml` file to customize the Spark deployment as needed.
- Check the logs and outputs for any errors or additional information.

[Back to Main README](../README.md)