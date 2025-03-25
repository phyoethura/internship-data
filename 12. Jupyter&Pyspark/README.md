# PySpark & Jupyter Notebooks Deployed On Kubernetes

## Installation and Deployment

## helm version - 8.7.2 ---- Spark versin - 3.5.0

### Install Spark via Helm Chart (Bitnami) 



```sh
$ helm repo add bitnami https://charts.bitnami.com/bitnami
$ helm search repo bitnami
$ helm install kayvan-release bitnami/spark --version 8.7.2
```

### Deploy Jupyter Workloads

Create a file named `jupyter.yaml` with the following content:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
   name: jupiter-spark
   namespace: spark
spec:
   replicas: 1
   selector:
      matchLabels:
         app: spark
   template:
      metadata:
         labels:
            app: spark
      spec:
         containers:
            - name: jupiter-spark-container
              image: docker.arvancloud.ir/jupyter/all-spark-notebook
              imagePullPolicy: IfNotPresent
              ports:
              - containerPort: 8888
              env: 
              - name: JUPYTER_ENABLE_LAB
                value: "yes"
---
apiVersion: v1
kind: Service
metadata:
   name: jupiter-spark-svc
   namespace: spark
spec:
   type: LoadBalancer
   selector:
      app: spark
   ports:
      - port: 8888
        targetPort: 8888
        
---
apiVersion: v1
kind: Service
metadata:
  name: jupiter-spark-driver-headless
  namespace: spark
spec:
  clusterIP: None
  selector:
    app: spark
```

Apply the configuration:

```sh
kubectl apply -f jupyter.yaml
```

Check the status of the pods and services:

```sh
kubectl get pod -n spark
kubectl get svc -n spark
```

### Spark Master URL

The Spark master URL address can be found using the following:

- `spark://kayvan-release-spark-master-0.kayvan-release-spark-headless.default.svc.cluster.local:7077`
- Or by using the External IP (e.g., `10.111.0.56`): `spark://10.111.0.56:7077`

### Accessing Jupyter Notebook

To open Jupyter Notebook, use the Jupyter service External IP (e.g., `10.111.0.55:8888`). Create a password using the provided token.

## Running PySpark Code on Jupyter Notebook

### Simple PySpark Application

Open a new Jupyter Notebook and write the following Python code. Press `Shift + Enter` to execute each block.

```python
from pyspark.sql import SparkSession
import socket

spark = SparkSession.builder.master("spark://wky-spark-master-0.wky-spark-headless.spark.svc.cluster.local:7077")\
            .appName("lucky-test2").config('spark.driver.host', socket.gethostbyname(socket.gethostname()))\
            .getOrCreate()

rdd = spark.sparkContext.parallelize([1,2,3,4,5])
print(rdd.count())
spark.stop()
```

### Concurrent PySpark Applications

To run multiple PySpark applications concurrently, use the following code:

```python
import multiprocessing
from random import random
from operator import add
from pyspark.sql import SparkSession
import socket

def calculate_pi(app_id, partitions=2):
    spark = SparkSession.builder \
        .master("spark://wky-spark-master-0.wky-spark-headless.spark.svc.cluster.local:7077") \
        .appName(f"PythonPiLeeNo.-{app_id}") \
        .config('spark.driver.host', socket.gethostbyname(socket.gethostname())) \
        .getOrCreate()

    n = 100000 * partitions

    def f(_: int) -> float:
        x = random() * 2 - 1
        y = random() * 2 - 1
        return 1 if x ** 2 + y ** 2 <= 1 else 0

    count = spark.sparkContext.parallelize(range(1, n + 1), partitions).map(f).reduce(add)
    print(f"[App {app_id}] Pi is roughly {4.0 * count / n}")

    spark.stop()

if __name__ == "__main__":
    processes = []
    
    for i in range(10):
        partitions = (i + 1) * 2  # Increasing partitions per instance
        process = multiprocessing.Process(target=calculate_pi, args=(i + 1, partitions))
        process.start()
        processes.append(process)

    for process in processes:
        process.join()
```

## Conclusion

By following the steps outlined in this guide, you can successfully deploy PySpark and Jupyter Notebooks on a Kubernetes cluster. This setup allows you to leverage the scalability and flexibility of Kubernetes for your data processing and analysis tasks using PySpark. The provided configurations for deploying Spark via Helm and setting up Jupyter workloads ensure that you can easily manage and access your Spark cluster and Jupyter environment. Additionally, running PySpark code and concurrent applications on Jupyter Notebooks demonstrates the practical use of this deployment in performing distributed data processing tasks efficiently.

[Back to Main README](../README.md)