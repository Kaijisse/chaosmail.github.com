---
layout: post
title: "Getting Started with Microsoft SQL 2019 Big Data clusters"
date: 2019-02-01 16:00:00
categories: hadoop
tags: bigdata hadoop spark sql2019
comments: true
---

Microsoft latest [SQL Server 2019][sql-server-2019] (preview) comes in a new version, the SQL Server 2019 Big Data cluster (BDC). There are a couple of cool things about the BDC version: 
(1) it runs on [Kubernetes][kubernetes]
(2) it integrates a sharded SQL engine
(3) it integrates [HDFS][apache-hdfs] (a distributed file storage)
(4) it integrates [Spark][apache-spark] (a distributed compute engine)
(5) and both services Spark and HDFS run behind an [Apache Knox][apache-knox] Gateway (HTTPS application gateway for Hadoop). 

On top, using Polybase you can connect to many different external data sources such as MongoDB, Oracle, Teradata, SAP Hana, and many more. Hence, SQL Server 2019 Big Data cluster (BDC) is a scalable, performant and maintainable SQL platform, Data Warehouse, Data Lake and Data Science platform without compromising cloud and on-premise. In this blog post I want to give you a quick start tutorial into [SQL 2019 Big Data clusters (BDC)][sql-server-2019-bigdata] and show you how to set it up on Azure Kubernetes Services (AKS), upload some data to HDFS and access the data from SQL and Spark.

## SQL Server 2019 Big Data cluster (BDC)

[SQL Server 2019 Big Data cluster (BDC)][sql-server-2019-bigdata] is one of the most exciting pieces of technologies I have seen in a long time. Here is why.

![SQL Server 2019 for Big Data architecture Source: [microsoft.com/sqlserver](https://cloudblogs.microsoft.com/sqlserver/2018/09/25/introducing-microsoft-sql-server-2019-big-data-clusters/)]({{ site.baseurl }}/images/sql2019/SQL-Server-2019-big-data-cluster.png "SQL Server 2019 for Big Data architecture"){: .image-col-1}

### Kubernetes
SQL Server 2019 builds on a new abstraction layer called *Platform Abstraction Layer* (PAL) which let's you run SQL Server on multiple platforms and environments, such as Windows, Linux, and Containers. To take this one step further, we can run SQL Server clusters entirely within Kubernetes - either locally (e.g. on [Minikube](https://kubernetes.io/docs/setup/minikube/)), on on-premise clusters or in the cloud (e.g. on Azure Kubernetes Services). All data is persisted using [*Persistent Volumes*](https://docs.microsoft.com/en-us/sql/big-data-cluster/concept-data-persistence?view=sqlallproducts-allversions). To facilitate operations, there is a new `mssqlctl` command to scaffold, configure, and scale SQL Server 2019 clusters in Kubernetes.

### SQL Master Instance
If you deploy SQL Server 2019 as a cluster in [Kubernetes][kubernetes], it comes with a SQL *Master Instance* and multiple SQL engine compute and storage shards. The great thing about the Master Instance is that it is just a normal SQL instance - you can use all existing tooling, code, etc. and interact with the SQL Server cluster as if it was a single DB instance. If you stream data to the cluster, you can stream the data directly to the SQL shards without going through the Master Instance. This gives you optimal throughput performance.

### Polybase
You might know *Polybase* from SQL Server 2016 as a service that let's you connect to flat HDFS data sources. With SQL Server 2019, you can now as well connect to relational data sources (e.g. Oracle, Teradata, SAP Hana, etc.) or NoSQL data sources (e.g. Mongo DB, Cosmos DB, etc.) as well as using Polybase and [external tables](https://docs.microsoft.com/en-us/sql/relational-databases/polybase/data-virtualization?view=sqlallproducts-allversions) - both with [predicate pushdown filters](https://blogs.msdn.microsoft.com/sql_server_team/predicate-pushdown-and-why-should-i-care/). It's a fantastic feature turning your SQL Server 2019 cluster into your central data hub.

### HDFS
Now comes the fun part. When you deploy a SQL Server 2019 BDC, you also deploy an [*Hadoop Distributed Filesystem* (HDFS)][apache-hdfs] within Kubernetes. With the [tiered storage feature in HDFS](https://www.microsoft.com/en-us/research/project/tiered-storage/) you can as well mount existing HDFS clusters into the integrated SQL Server 2019 HDFS. Using the [integrated Polybase scale-out groups](https://docs.microsoft.com/en-us/sql/relational-databases/polybase/configure-scale-out-groups-windows?view=sqlallproducts-allversions) you can efficiently access this distributed data from SQL with [external tables](https://docs.microsoft.com/en-us/sql/relational-databases/polybase/data-virtualization-csv?view=sqlallproducts-allversions). If you install SQL Server 2019 as a BDC, all the configurations of those services is done automatically, even pass-through authentication. These features allows your SQL Server 2019 cluster to become the central data storage for both relational structured and massive volumes of flat unstructured data.

### Spark
And it's getting better. The SQL Server 2019 BDC also includes a [*Spark*][apache-spark] run-time co-located with the HDFS data pools. For me - coming from a Big Data background - this is huge! This means, you can take advantage of all Spark features (SparkSQL, Dataframes, MLlib for machine learning, GraphX for graph processing, Structured Streaming for stream processing, and much more) directly within your SQL cluster. Now, your SQL 2019 cluster can as well be used by your data scientists and data engineers as a central Big Data hub. Thanks to integration of [Apache Livy](https://livy.incubator.apache.org/) (a Spark Rest Gateway) you can utilize this functionality with your existing tooling, such as Jupyter or Zeppelin notebooks out-of-the-box.

### Much More ... (Knox, Grafana, SSIS, Report Server, etc.)
Once we are running in Kubernetes, you can as well add many more services to the cluster and manage, operate, and scale them together. The Spark and HDFS functionality is configured with an [Apache Knox Gateway][apache-knox] (HTTPS application gateway for Hadoop) and can be integrated into many other existing services (e.g. processes writing to HDFS, etc.). SQL Server 2019 BDC ships comes with an  integrated [Cluster Configuration portal](https://docs.microsoft.com/en-us/sql/big-data-cluster/cluster-admin-portal?view=sqlallproducts-allversions) and a Grafana dashboard for monitoring all relevant service metrics.

Deploying other co-located services to the same Kubernetes cluster becomes quite easy. Services such as Integration Services, Analysis Services or Report Server can simply be deployed and scaled to the same SQL Server 2019 cluster as additional Kubernetes pods.

Another cool feature of SQL Server 2019 worth mentioning is that along Python and R it will also support User Defined Functions (UDFs) written in Java. Niels Berglund has many examples in his [Blog post series](http://www.nielsberglund.com/s2k19_ext_framework_java/).

## Installation

Currently, SQL Server 2019 and SQL Server 2019 Big data cluster (BDC) are still in private preview. Hence, you need to apply for the [Early Adoption Program][sql-server-2019-early-adoption] which will grant you access to Microsoft's private registry and SQL Server 2019 images. You are also assigned a buddy (a PM on the SQL Server 2019 team) as well as granted access to a private Teams channel. Hence, if you want to try it already today, you should definitely sign up!

In this section we will go through the prerequisites and installation process as documented in the [SQL Server 2019 installation guidelines][sql-server-2019-deploy-bigdata] for Big Data analytics. In the documentation, you will find a link to a [Python script][sql-server-2019-deploy-bigdata-github] that allows you to spin up SQL 2019 on Azure Kubernetes Services (AKS).

> If you want to install SQL Server 2019 BDC on your on-premise Kubernetes cluster, you can follow the steps in [Christopher Adkin's Blog](https://chrisadkin.io/2018/12/18/building-a-kubernetes-cluster-for-sql-server-2019-big-data-clusters-part-1-hyper-v-virtual-machine-creation/
). You can find an official deployment guide for BDC on [Minikube in the Microsoft docs](https://docs.microsoft.com/en-us/sql/big-data-cluster/deploy-on-minikube?view=sql-server-ver15).

### Prerequisites: Kubernetes and MSSQL clients

To [deploy a SQL Server 2019 Big Data cluster (BDC)][sql-server-2019-deploy-bigdata] on Azure Kubernetes Services (AKS), you need the following tools installed. For this tutorial, I installed all these tools on Ubuntu 18.04 LTS on [WSL](https://docs.microsoft.com/en-us/windows/wsl/install-win10) (Windows Subsystem for Linux).

* [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)
* [kubectl][install-kubectl]
* [mssqlctl][install-mssqlctl]
* [SQL Server 2019 Early Adoption Program][sql-server-2019-early-adoption]

To avoid any problems with Kubernetes APIs, it's best to install the same `kubectl` version as the Kubernetes version on AKS. In the SQL Server 2019 docs, the version `1.10.9` is recommended. To get all versions of supported Kubernetes versions, run the following snippet.

```sh
$ az aks get-versions --location westeurope --output table
KubernetesVersion    Upgrades
-------------------  -----------------------
1.12.4               None available
1.11.6               1.12.4
1.11.5               1.11.6, 1.12.4
1.10.12              1.11.5, 1.11.6
1.10.9               1.10.12, 1.11.5, 1.11.6
1.9.11               1.10.9, 1.10.12
1.9.10               1.9.11, 1.10.9, 1.10.12
1.8.15               1.9.10, 1.9.11
1.8.14               1.8.15, 1.9.10, 1.9.11
```

Hence, in this case we also [install][install-kubectl] the Kubernetes `1.10.9` client.

```sh
sudo apt-get update && sudo apt-get install -y apt-transport-https
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubectl=1.10.9-00
```

The `mssqlctl` tool is a handy command-line utility that allows you to create and manage SQL Server 2019 Big Data cluster installations. You can [install it][install-mssqlctl] using pip with the following command:

```sh
$ pip3 install --extra-index-url https://private-repo.microsoft.com/python/ctp-2.2 mssqlctl --user
```

Before you continue, make sure that both `kubectl` and `mssqlctl` commands are available. If they are not, you might restart the current bash session.

### Prerequisites: Azure Data Studio

[Azure Data Studio][azure-data-studio] is a cross-platform management tool for Microsoft databases. It's like SQL Server Management Studio on top of the popular VS Code editor engine, a rich T-SQL editor with IntelliSense and Plugin support. Currently, it's the easiest way to connect to the different SQL Server 2019 endpoints (SQL, HDFS, and Spark). To do so, you need to [install Data Studio][azure-data-studio-install] and the [SQL Server 2019 extension][azure-data-studio-install-sql2019].

The following screenshot (Source: [Microsoft/azuredatastudio](https://github.com/Microsoft/azuredatastudio)) shows an overview of Azure Data Studio and its capabilities.

![Azure Data Studio overview]({{ site.baseurl }}/images/sql2019/data-studio-overview.jpg "Azure Data Studio overview"){: .image-col-1}

Azure Data Studio also supports Jupyter-style notebooks for T-SQL and Spark. The following screenshot shows Data Studio with the notebooks extension.

![Azure Data Studio with SQL Server 2019 extension]({{ site.baseurl }}/images/sql2019/data-studio.png "Azure Data Studio with SQL Server 2019 extension"){: .image-col-1}

### Install SQL Server 2019 BDC on Azure Kubernetes Services (AKS)

In this section, we will follow the steps from the [installation script][sql-server-2019-deploy-bigdata-github] in order to install SQL Server 2019 for Big Data on AKS. I will give you a bit more details and explanation about the executed steps. If you just want to install SQL Server 2019 for Big Data, you can as well just use the script.

First, we start setting all required parameters for the installation process. For the docker username and password, please use the credentials provided in the Early Adoption program. These will give you access to Microsoft's internal registry with the latest SQL Server 2019 images.

```sh
# Provide your Azure subscription ID
SUBSCRIPTION_ID="***"
# Provide Azure resource group name to be created
GROUP_NAME="demos.sql2019"

# Provide Azure region
AZURE_REGION="westeurope"
# Provide VM size for the AKS cluster
VM_SIZE="Standard_L4s"
# Provide number of worker nodes for AKS cluster
AKS_NODE_COUNT="3"
# Provide Kubernetes version
KUBERNETES_VERSION="1.10.9"

# This is both Kubernetes cluster name and SQL Big Data cluster name
# Provide name of AKS cluster and SQL big data cluster
CLUSTER_NAME="sqlbigdata"
# Provide username to be used for Controller user
CONTROLLER_USERNAME="admin"
# This password will be use for Controller user, Knox user and SQL Server Master SA accounts
# Provide password to be used for Controller user, Knox user and SQL Server Master SA accounts
PASSWORD="MySQLBigData2019"
CONTROLLER_PASSWORD="$PASSWORD"
MSSQL_SA_PASSWORD="$PASSWORD"
KNOX_PASSWORD="$PASSWORD"

# Private Microsoft registry
DOCKER_REGISTRY="private-repo.microsoft.com"
DOCKER_REPOSITORY="mssql-private-preview"
DOCKER_IMAGE_TAG="latest"

# Provide your Docker username
DOCKER_USERNAME="***"
# Provide your Docker password
DOCKER_PASSWORD="***"
```

Now, we can go ahead and create the AKS cluster.

```sh
$ az aks create --name $CLUSTER_NAME --resource-group $GROUP_NAME --generate-ssh-keys \
    --node-vm-size $VM_SIZE --node-count $AKS_NODE_COUNT --kubernetes-version $KUBERNETES_VERSION
```

If you run this script for the first time, you can skip the following step. However, if you create multiple AKS clusters with the same name, you first need to clean the credentials.

```sh
$ kubectl config unset "clusters.$CLUSTER_NAME"
$ kubectl config unset "users.clusterAdmin_${GROUP_NAME}_${CLUSTER_NAME}"
```
In the next step, we retrieve the credentials for the cluster. This will register the credentials in the `kubectl` config.

```sh
$ az aks get-credentials --name $CLUSTER_NAME --resource-group $GROUP_NAME --admin
```

In order to access the Kubernetes dashboard, we also need to create a role binding. I took this line from [Pascal Naber's blog post](https://pascalnaber.wordpress.com/2018/06/17/access-dashboard-on-aks-with-rbac-enabled/).

```sh
$ kubectl create clusterrolebinding kubernetes-dashboard -n kube-system \
    --clusterrole=cluster-admin --serviceaccount=kube-system:kubernetes-dashboard
```

Next, we can open the Kuerbenetes dashboard for the newly created AKS cluster and see if everything looks fine. To do so, we can forward the required ports to localhost.

```sh
$ az aks browse --resource-group $GROUP_NAME --name $CLUSTER_NAME
```

The Kubernetes dashboard should now be available via [http://localhost:8001](http://localhost:8001). I recommend you to open it and take a look at your newly created cluster.

Finally, we can deploy SQL Server 2019 BDC on the Kubernetes cluster using the `mssqlctl` command-line utility.

```sh
$ mssqlctl create cluster $CLUSTER_NAME
```

Great, that was it! You are now ready to get started. The following figure shows the Kubernetes dashboard with an installed instance of SQL Server 2019 BDC. You can see the Storage, Data and Compute pools as well as the SQL Master instance.

![Kubernetes dashboard for SQL Server 2019 BDC]({{ site.baseurl }}/images/sql2019/kubernetes.png "Kubernetes dashboard for SQL Server 2019"){: .image-col-1}

## Querying SQL Server 2019 BDC

For this section, we will use Azure Data Studio with the SQL Server 2019 extension which let's us connect to both the SQL Server endpoint as well as the Knox endpoint for HDFS and Spark.

### Working with HDFS

First, we will put some data into the Big Data cluster. Let's retrieve the Knox URL for HDFS.

```sh
$ kubectl get service service-security-lb -o=custom-columns="IP:.status.loadBalancer.ingress[0].ip,PORT:.spec.ports[0].port" -n $CLUSTER_NAME
```

Using this URL, we can build the WebHDFS URL and use any HDFS client to connect to the file system.

```
https://<service-security-lb service external IP address>:30433/gateway/default/webhdfs/v1/
```

You can follow the [guidelines in the Microsoft docs](https://docs.microsoft.com/en-us/sql/big-data-cluster/data-ingestion-curl?view=sqlallproducts-allversions) using `curl` or simply use the integrated HDFS explorer in Data Studio. To do so, you must [create a new connection in Data Studio](https://docs.microsoft.com/en-us/sql/big-data-cluster/connect-to-big-data-cluster?view=sqlallproducts-allversions) and select `SQL Server Big Data Cluster`.

![HDFS in SQL Server 2019 (Source: [docs.microsoft.com](https://docs.microsoft.com/en-us/sql/big-data-cluster/connect-to-big-data-cluster?view=sqlallproducts-allversions))]({{ site.baseurl }}/images/sql2019/connect-data-services-node.png "HDFS in SQL Server 2019"){: .image-col-1}

In order to query the HDFS data from SQL, you can configure external tables with the [external table wizard](https://docs.microsoft.com/en-us/sql/relational-databases/polybase/data-virtualization-csv?toc=%2fsql%2fbig-data-cluster%2ftoc.json&view=sql-server-ver15).

### Working with SQL

Let's retrieve the SQL Server *Master Instance* endpoint from the Kubernetes cluster.

```sh
$ kubectl get service endpoint-master-pool -o=custom-columns="IP:.status.loadBalancer.ingress[0].ip,PORT:.spec.ports[0].port" -n $CLUSTER_NAME
```

It is a just a normal SQL Server master instance like in SQL Server 2017. Hence, you can connect to the SQL Server endpoint using standard SQL tooling, such as SQL Server Management Studio or Azure Data Studio. In Data Studio, select connection type `Microsoft SQL Server`.

You can verify yourself that this is a standard SQL Server instance. The following screenshot shows a query over an external table storing data in HDFS on the same cluster.

![External table in SQL Server 2019]({{ site.baseurl }}/images/sql2019/sql.png "External table in SQL Server 2019"){: .image-col-1}

Using Polybase, you can as well setup external tables to many other relational data sources, such as Oracle and SAP Hana. Using the [external table wizard](https://docs.microsoft.com/en-us/sql/relational-databases/polybase/data-virtualization?toc=%2fsql%2fbig-data-cluster%2ftoc.json&view=sql-server-ver15) in Data Studio this connection is easy to setup.

> You can find many more demos for SQL Server 2019 on [Bob Ward's Github repository](https://github.com/Microsoft/bobsql).

### Working with Spark

To work with Spark in SQL Server 2019 BDC, we can leverage the notebook capabilities of Data Studio. Once we connected to the Big Data cluster, we will see options to create Spark notebooks for this instance.

In the current version, the credentials from Spark are not yet passed to the SQL engine automatically. Hence we have to supply a username and password along with the local database host to build the JDBC connection string. Here is a simple PySpark script to connect to the SQL Server database from within Spark.

```python
host = ""
database = "demos"
user = "sa"
password = ""
table = "dbo.NYCTaxiTrips"
jdbc_url = "jdbc:sqlserver://%s;database=%s;user=%s;password=%s" % (host, database, user, password)

df = spark.read.format("jdbc") \
       .option("url", jdbc_url) \
       .option("dbtable", table) \
       .load()

df.show()
```

The following screenshot shows the above query executed on my SQL Server 2019 BDC instance on the NYC Taxi Trips dataset.

![Spark accessing SQL in SQL Server 2019]({{ site.baseurl }}/images/sql2019/spark.png "Spark accessing SQL in SQL Server 2019"){: .image-col-1}

If you need to [install additional Python packages](https://jakevdp.github.io/blog/2017/12/05/installing-python-packages-from-jupyter/) on the cluster nodes or [configure the Spark environment](https://becominghuman.ai/setting-up-a-scalable-data-exploration-environment-with-spark-and-jupyter-lab-22dbe7046269), you can use the Jupyter magic commands.

I am sure you can see why this is really cool, right? You can easily run your Spark ETL, pre-processing and Machine Learning pipelines on data both stored in SQL and HDFS or any external sources.

> You can find many more Big Data samples on [Buck Woody's Github repository](https://github.com/Microsoft/sqlworkshops/tree/master/sqlserver2019bigdataclusters).

## Summary



## Resources

* [SQL Server 2019][sql-server-2019]
* [SQL Server 2019 Big Data cluster (BDC)][sql-server-2019-bigdata]
* [Kubernetes][kubernetes]
* [Hadoop Distributed Filesystem (HDFS)][apache-hdfs]
* [Apache Spark][apache-spark]
* [Apache Knox][apache-knox]
* [SQL Server 2019 Early Adoption Program][sql-server-2019-early-adoption]
* [SQL Server 2019 BDC installation guide][sql-server-2019-deploy-bigdata]
* [SQL Server 2019 BDC installation script][sql-server-2019-deploy-bigdata-github]
* [Kubectl installation guide][install-kubectl]
* [Mssqlctl installation guide][install-mssqlctl]
* [Azure Data Studio][azure-data-studio]
* [SQL Workshops][sql-workshops]

> Thanks to [Kaijisse Waaijer](https://www.linkedin.com/in/kaijisse-w-85304ba0/).

[sql-server-2019]: https://www.microsoft.com/en-us/sql-server/sql-server-2019
[sql-server-2019-bigdata]: https://docs.microsoft.com/en-us/sql/big-data-cluster/big-data-cluster-overview?view=sql-server-ver15
[kubernetes]: https://kubernetes.io/
[apache-hdfs]: https://hadoop.apache.org/docs/current1/hdfs_design.html#Introduction
[apache-spark]: http://spark.apache.org/
[apache-knox]: https://knox.apache.org/
[sql-server-2019-early-adoption]: https://sqlservervnexteap.azurewebsites.net/
[sql-server-2019-deploy-bigdata]: https://docs.microsoft.com/en-us/sql/big-data-cluster/quickstart-big-data-cluster-deploy?view=sql-server-ver15
[sql-server-2019-deploy-bigdata-github]: https://github.com/Microsoft/sql-server-samples/blob/master/samples/features/sql-big-data-cluster/deployment/aks/deploy-sql-big-data-aks.py
[install-kubectl]: https://kubernetes.io/docs/tasks/tools/install-kubectl/
[install-mssqlctl]: https://docs.microsoft.com/en-us/sql/big-data-cluster/deploy-install-mssqlctl?view=sqlallproducts-allversions

[azure-data-studio]: https://docs.microsoft.com/en-us/sql/azure-data-studio/what-is?view=sqlallproducts-allversions
[azure-data-studio-install]: https://docs.microsoft.com/en-us/sql/azure-data-studio/download?view=sqlallproducts-allversions
[azure-data-studio-install-sql2019]: https://docs.microsoft.com/en-us/sql/azure-data-studio/sql-server-2019-extension?view=sqlallproducts-allversions
[sql-workshops]: https://microsoft.github.io/sqlworkshops/
