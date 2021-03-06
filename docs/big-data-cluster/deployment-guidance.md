---
title: How to deploy SQL Server big data clusters on Kubernetes | Microsoft Docs
description: Learn how to deploy SQL Server 2019 big data clusters (preview) on Kubernetes.
author: rothja 
ms.author: jroth 
manager: craigg
ms.date: 10/08/2018
ms.topic: conceptual
ms.prod: sql
---

# How to deploy SQL Server big data cluster on Kubernetes

SQL Server big data cluster can be deployed as docker containers on a Kubernetes cluster. This is an overview of the setup and configuration steps:

- Set up Kubernetes cluster on a single VM, cluster of VMs, or in Azure Kubernetes Service (AKS).
- Install the cluster configuration tool **mssqlctl** on your client machine.
- Deploy SQL Server big data cluster in a Kubernetes cluster.

[!INCLUDE [Limited public preview note](../includes/big-data-cluster-preview-note.md)]

## <a id="prereqs"></a> Kubernetes cluster prerequisites

SQL Server big data cluster requires a minimum v1.10 version for Kubernetes, for both server and client. To install a specific version on kubectl client, see [Install kubectl binary via curl](https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl).  Latest versions of minikube and AKS are at least 1.10. For AKS, you will need to use `--kubernetes-version` parameter to specify a version different than default.

> [!NOTE]
> Note that the client and server Kubernetes versions should be +1 or -1 minor version. For more information, see [Kubernetes supported releases and component skew](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/release/versioning.md#supported-releases-and-component-skew).

## <a id="kubernetes"></a> Kubernetes cluster setup

If you already have a Kubernetes cluster that meets above prerequisites, then you can skip directly to the [deployment step](#deploy). This section assumes a basic understanding of Kubernetes concepts.  For detailed information on Kubernetes, see the [Kubernetes documentation](https://kubernetes.io/docs/home).

You can choose to deploy Kubernetes in any of three ways:

| Deploy Kubernetes on: | Description |
|---|---|
| **Minikube** | A single-node Kubernetes cluster in a VM. |
| **Azure Kubernetes Services (AKS)** | A managed Kubernetes container service in Azure. |
| **Multiple machines** | A Kubernetes cluster deployed on physical or virtual machines using **kubeadm** |

For guidance on configuring one of these Kubernetes cluster options for a SQL Server big data cluster, see one of the following articles:

   - [Configure Minikube](deploy-on-minikube.md)
   - [Configure Kubernetes on Azure Kubernetes Service](deploy-on-aks.md)
   - [Configure Kubernetes on multiple machines with kubeadm](deploy-with-kubeadm.md)
   
> [!TIP]
> For a sample python script that deploys both AKS and SQL Server big data cluster, see [Deploy a SQL Server big data cluster on Azure Kubernetes Service (AKS)](https://github.com/Microsoft/sql-server-samples/tree/master/samples/features/sql-big-data-cluster/deployment/aks).

## <a id="deploy"></a> Deploy SQL Server big data cluster

After you have configured your Kubernetes cluster, you can proceed with the deployment for SQL Server big data cluster. To deploy a big data cluster in Azure with all default configurations for a dev/test environment, follow the instructions in this article:

[Quickstart: Deploy SQL Server big data cluster on Kubernetes](quickstart-big-data-cluster-deploy.md)

If you want to customize your big data cluster configuration according to your workload needs, follow the next set of instructions.

## Verify kubernetes configuration

Run the **kubectl** command to view the cluster configuration. Ensure that kubectl is pointed to the correct cluster context.

```bash
kubectl config view
```

## <a id="mssqlctl"></a> Install mssqlctl

**mssqlctl** is a command-line utility written in Python that enables cluster administrators to bootstrap and manage the big data cluster via REST APIs. The minimum Python version required is v3.5. You must also have `pip` that is used to download and install **mssqlctl** tool. 

### Windows mssqlctl installation

1. On a Windows client, download the necessary Python package from [https://www.python.org/downloads/](https://www.python.org/downloads/). For python3.5.3 and later, pip3 is also installed when you install Python. 

   > [!TIP] 
   > When installing Python3, select to add Python to your path. If you do not, you can later find where pip3 is located and manually add it to your path.

1. Make sure that you have the latest **requests** package.

   ```cmd
   python -m pip install requests
   python -m pip install requests --upgrade
   ```

1. Install **mssqlctl** with the following command:

   ```bash
   pip3 install --index-url https://private-repo.microsoft.com/python/ctp-2.0 mssqlctl
   ```

### Linux mssqlctl installation

On Linux, you must install the **python3** and **python3-pip** packages and then run `sudo pip3 install --upgrade pip`. This installs the latest 3.5 version of Python and pip. The following example shows how these commands would work for Ubuntu (if you are using another platform, modify the commands for your package manager):

1. Install the necessary Python packages:

   ```bash
   sudo apt-get update && /
   sudo apt-get install -y python3 && /
   sudo apt-get install -y python3-pip && /
   sudo -H pip3 install --upgrade pip
   ```

1. Make sure that you have the latest **requests** package.

   ```bash
   sudo -H python3 -m pip install requests
   sudo -H python3 -m pip install requests --upgrade
   ```

1. Install **mssqlctl** with the following command:

   ```bash
   pip3 install --index-url https://private-repo.microsoft.com/python/ctp-2.0 mssqlctl
   ```

## Define environment variables

The cluster configuration can be customized using a set of environment variables that are passed to the `mssqlctl create cluster` command. Most of the environment variables are optional with default values as per below. Note that there are environment variables like credentials that require user input.

| Environment variable | Required | Default value | Description |
|---|---|---|---|
| **ACCEPT_EULA** | Yes | N/A | Accept the SQL Server license agreement (for example, 'Y').  |
| **CLUSTER_NAME** | Yes | N/A | The name of the Kubernetes namespace to deploy SQLServer big data cluster into. |
| **CLUSTER_PLATFORM** | Yes | N/A | The platform the Kubernetes cluster is deployed. Can be `aks`, `minikube`, `kubernetes`|
| **CLUSTER_COMPUTE_POOL_REPLICAS** | No | 1 | The number of compute pool replicas to build out. In CTP2.0 only valued allowed is 1. |
| **CLUSTER_DATA_POOL_REPLICAS** | No | 2 | The number of data pool replicas to build out. |
| **CLUSTER_STORAGE_POOL_REPLICAS** | No | 2 | The number of storage pool replicas to build out. |
| **DOCKER_REGISTRY** | Yes | TBD | The private registry where the images used to deploy the cluster are stored. |
| **DOCKER_REPOSITORY** | Yes | TBD | The private repository within the above registry where images are stored.  It is required for the duration of the gated public preview. |
| **DOCKER_USERNAME** | Yes | N/A | The username to access the container images in case they are stored in a private repository. It is required for the duration of the gated public preview. |
| **DOCKER_PASSWORD** | Yes | N/A | The password to access the above private repository. It is required for the duration of the gated public preview.|
| **DOCKER_EMAIL** | Yes | N/A | The email associated with the above private repository. It is required for the duration of the gated private preview. |
| **DOCKER_IMAGE_TAG** | No | latest | The label used to tag the images. |
| **DOCKER_IMAGE_POLICY** | No | Always | Always force a pull of the images.  |
| **DOCKER_PRIVATE_REGISTRY** | Yes | 1 | For the timeframe of the gated public preview, this value has to be set to 1. |
| **CONTROLLER_USERNAME** | Yes | N/A | The username for the cluster administrator. |
| **CONTROLLER_PASSWORD** | Yes | N/A | The password for the cluster administrator. |
| **KNOX_PASSWORD** | Yes | N/A | The password for Knox user. |
| **MSSQL_SA_PASSWORD** | Yes | N/A | The password of SA user for SQL master instance. |
| **USE_PERSISTENT_VOLUME** | No | true | `true` to use Kubernetes Persistent Volume Claims for pod storage.  `false` to use ephemeral host storage for pod storage. See the [data persistence](concept-data-persistence.md) article for more details. If you deploy SQL Server big data cluster on minikube and USE_PERSISTENT_VOLUME=true, you must set the value for `STORAGE_CLASS_NAME=standard`. |
| **STORAGE_CLASS_NAME** | No | default | If `USE_PERSISTENT_VOLUME` is `true` this indicates the name of the Kubernetes Storage Class to use. See the [data persistence](concept-data-persistence.md) article for more details. If you deploy SQL Server big data cluster on minikube, the default storage class name is different and you must override it by setting `STORAGE_CLASS_NAME=standard`. |
| **MASTER_SQL_PORT** | No | 31433 | The TCP/IP port that the master SQL instance listens on the public network. |
| **KNOX_PORT** | No | 30443 | The TCP/IP port that Apache Knox listens on the public network. |
| **GRAFANA_PORT** | No | 30888 | The TCP/IP port that the Grafana monitoring application listens on the public network. |
| **KIBANA_PORT** | No | 30999 | The TCP/IP port that the Kibana log search application listens on the public network. |

> [!IMPORTANT]
>1. For the duration of the limited private preview, credentials for the private Docker registry will be provided to you upon triaging your [EAP registration](https://aka.ms/eapsignup).
>1. For an on-premises cluster built with **kubeadm**, the value for environment variable `CLUSTER_PLATFORM` is `kubernetes`. Also, when `USE_PERSISTENT_VOLUME=true`, you must pre-provision a Kubernetes storage class and pass it through using the `STORAGE_CLASS_NAME`.
>1. Make sure you wrap the passwords in double quotes if it contains any special characters. You can set the MSSQL_SA_PASSWORD to whatever you like, but make sure they are sufficiently complex and don???t use the `!`, `&` or `???` characters. Note that double quotes delimiters work only in bash commands.
>1. The name of your cluster must be only lower case alpha-numeric characters, no spaces. All Kubernetes artifacts (containers, pods, statefull sets, services) for the cluster will be created in a namespace with same name as the cluster name specified.
>1. The **SA** account is a system administrator on the SQL Server Master instance that gets created during setup. After creating your SQL Server container, the MSSQL_SA_PASSWORD environment variable you specified is discoverable by running echo $MSSQL_SA_PASSWORD in the container. For security purposes, change your SA password as per best practices documented [here](https://docs.microsoft.com/sql/linux/quickstart-install-connect-docker?view=sql-server-2017#change-the-sa-password).

Setting the environment variables required for deploying a big data cluster differs depending on whether you are using Windows or Linux client.  Choose the steps below depending on which operating system you are using.

Initialize the following environment variables, they are required for deploying the cluster:

### Windows

Using a CMD window (not PowerShell), configure the following environment variables. Do not use quotes around the values.

```cmd
SET ACCEPT_EULA=Y
SET CLUSTER_PLATFORM=<minikube or aks or kubernetes>

SET CONTROLLER_USERNAME=<controller_admin_name ??? can be anything>
SET CONTROLLER_PASSWORD=<controller_admin_password ??? can be anything, password complexity compliant>
SET KNOX_PASSWORD=<knox_password ??? can be anything, password complexity compliant>
SET MSSQL_SA_PASSWORD=<sa_password_of_master_sql_instance, password complexity compliant>

SET DOCKER_REGISTRY=private-repo.microsoft.com
SET DOCKER_REPOSITORY=mssql-private-preview
SET DOCKER_USERNAME=<your username, credentials provided by Microsoft>
SET DOCKER_PASSWORD=<your password, credentials provided by Microsoft>
SET DOCKER_EMAIL=<your Docker email, use the username provided by Microsoft>
SET DOCKER_PRIVATE_REGISTRY="1"
```

### Linux

Initialize the following environment variables. In bash, you can use quotes around each value.

```bash
export ACCEPT_EULA=Y
export CLUSTER_PLATFORM=<minikube or aks or kubernetes>

export CONTROLLER_USERNAME="<controller_admin_name ??? can be anything>"
export CONTROLLER_PASSWORD="<controller_admin_password ??? can be anything, password complexity compliant>"
export KNOX_PASSWORD="<knox_password ??? can be anything, password complexity compliant>"
export MSSQL_SA_PASSWORD="<sa_password_of_master_sql_instance, password complexity compliant>"

export DOCKER_REGISTRY="private-repo.microsoft.com"
export DOCKER_REPOSITORY="mssql-private-preview"
export DOCKER_USERNAME="<your username, credentials provided by Microsoft>"
export DOCKER_PASSWORD="<your password, credentials provided by Microsoft>"
export DOCKER_EMAIL="<your Docker email, use the username provided by Microsoft>"
export DOCKER_PRIVATE_REGISTRY="1"
```

### Minikube settings

If you are deploying on minikube and `USE_PERSISTENT_VOLUME=true` (default), you must also override the default value for `STORAGE_CLASS_NAME` environment variable.

Use the following command on Windows for minikube deployments:

```cmd
SET STORAGE_CLASS_NAME=standard
```

Use the following command on Linux for minikube deployments:

```bash
export STORAGE_CLASS_NAME=standard
```

Alternatively, you can suppress using persistent volumes on minikube by setting `USE_PERSISTENT_VOLUME=false`.

### Kubadm settings

If you are deploying with kubeadm on your own physical or virtual machines, you must pre-provision a Kubernetes storage class and pass it through using the `STORAGE_CLASS_NAME`. Alternatively, you can suppress using persistent volumes by setting `USE_PERSISTENT_VOLUME=false`. For more information about persistent storage, see [Data persistence with SQL Server big data cluster on Kubernetes](concept-data-persistence.md).

## Deploy SQL Server big data cluster

The create cluster API is used to initialize the Kubernetes namespace and deploy all the application pods into the namespace. To deploy SQL Server big data cluster on your Kubernetes cluster, run the following command:

```bash
mssqlctl create cluster <name of your cluster>
```

During cluster bootstrap, the client command window will output the deployment status. You can also check the deployment status by running these commands in a different cmd window:

```bash
kubectl get all -n <name of your cluster>
kubectl get pods -n <name of your cluster>
kubectl get svc -n <name of your cluster>
```

You can see a more granular status and configuration for each pod by running:
```bash
kubectl describe pod <pod name> -n <name of your cluster>
```

Once the Controller pod is running, you can leverage the Deployment tab in the Cluster Administration Portal to monitor the deployment.

## <a id="masterip"></a> Get the SQL Server Master instance and SQL Server big data cluster IP addresses

After the deployment script has completed successfully, you can obtain the IP address of the SQL Server master instance using the steps outlined below. You will use this IP address and port number 31433 to connect to the SQL Server master instance (for example: **\<ip-address\>,31433**). Similarly, for the SQL Server big data cluster IP. All cluster endpoints are outlined in the Service Endpoints tab in the Cluster Administration Portal as well. You can use the Cluster Administration Portal to monitor the deployment. You can access the portal using the external IP address and port number for the `service-proxy-lb` (for example: **https://\<ip-address\>:30777**). Credentials for accessing the admin portal are the values of `CONTROLLER_USERNAME` and `CONTROLLER_PASSWORD` environment variables provided above.

### AKS

If you are using AKS, Azure provides the Azure LoadBalancer service. Run following command:

```bash
kubectl get svc service-master-pool-lb -n <name of your cluster>
kubectl get svc service-security-lb -n <name of your cluster>
kubectl get svc service-proxy-lb -n <name of your cluster>
```

Look for the **External-IP** value that is assigned to the service. Then, connect to the SQL Server master instance using the IP address at port 31433 (Ex: **\<ip-address\>,31433**) and to SQL Server big data cluster endpoint using the external-IP for `service-security-lb` service. 

### Minikube

If you are using Minikube, you need to run the following command to get the IP address you need to connect to. In addition to the IP, specify the port for the endpoint you need to connect to. To get all the service endpoints for 

```bash
minikube ip
```

Irrespective of the platform you are running your Kubernetes cluster on, to get all the service endpoints deployed for the cluster, run following command:
```bash
kubectl get svc -n <name of your cluster>
```

## Next steps

After successfully deploying SQL Server big data cluster to Kubernetes, [install the big data tools](deploy-big-data-tools.md) and try out some of the new capabilities and learn [How to use notebooks in SQL Server 2019 preview](notebooks-guidance.md).
