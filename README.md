# Setup Jupyetr Notebook with Spark Executors

## Download Spark binaries
```sh
    wget http://archive.apache.org/dist/spark/spark-2.4.4/spark-2.4.4-bin-hadoop2.7.tgz
    tar xvzf spark-2.4.4-bin-hadoop2.7.tgz
    rm spark-2.4.4-bin-hadoop2.7.tgz

    # Create symlink for easy docker build
    ln -s spark-2.4.4-bin-hadoop2.7 spark
```

## Build and push docker images
```sh
    cd spark && bin/docker-image-tool.sh -r docker.io/mdstech -t v2.4.0 build
    docker push mdstech/spark-py:v2.4.0
```

```sh
    # Build docker image for Jupyter notebook with Spark installation
    docker build -t mdstech/pyspark-notebook:v2.4.0 -f jupyter/Dockerfile .
    docker push mdstech/pyspark-notebook:v2.4.0
```

## Deploy Jupyter notebook into Cluster
```sh
    # Jupyter service is exposed as Nodeport for minikube use, remove for actual deployment
    kubectl apply -f jupyter-full.yaml
```

## Open Jupyter in browser
```sh
    minikube ip
    http://<minikube ip>/30001/jupyter?token=<from pod>
```

## Create sample spark notebook

### Create Spark Context with SQL Session
```python
    from pyspark import SparkContext, SparkConf
    from pyspark.sql import SparkSession
    import pyspark
    conf = pyspark.SparkConf()
    conf.setMaster("k8s://https://kubernetes.default.svc.cluster.local:443")
    conf.set("spark.kubernetes.container.image", "mdstech/spark-py:v2.4.0")
    conf.set("spark.kubernetes.authenticate.caCertFile", "/var/run/secrets/kubernetes.io/serviceaccount/ca.crt")
    conf.set("spark.kubernetes.authenticate.oauthTokenFile", "/var/run/secrets/kubernetes.io/serviceaccount/token")
    conf.set("spark.kubernetes.authenticate.driver.serviceAccountName", "my-pyspark-notebook")
    conf.set("spark.executor.instances", "1")
    conf.set("spark.driver.cores", "200m")
    conf.set("spark.kubernetes.executor.docker.image", "mdstech/spark-py:v2.4.0")
    conf.set("spark.driver.host", "my-pyspark-notebook-spark-driver.my-pyspark-notebook.svc.cluster.local")
    conf.set("spark.kubernetes.namespace", "my-pyspark-notebook")
    sc = pyspark.SparkContext(conf=conf)
```
### Step 2:
```python
    l = [('Alice', 1),('Bob', 3)]
    spark = SparkSession.builder.appName('my-spark-pi').getOrCreate()
```

### Step 3:
```python
    df = spark.createDataFrame(l, ['name', 'age'])
    df.filter(df.age > 1).collect()
```

### Step 4:
```python
    rdd = sc.parallelize(range(100000000))
    rdd.sum()
```

### Step 5:
```python
    sc.stop()
```

## Reference: [Thanks to Kublr team article](https://kublr.com/blog/running-spark-with-jupyter-notebook-hdfs-on-kubernetes/)
