# Deploying a Sample Nginx on Huawei Cloud CCE

This guide will walk you through creating a Cloud Container Engine (CCE) on Huawei Cloud and deploying a simple Nginx web server.

## Prerequisites

Before you start, ensure you have:

- A **Huawei Cloud** account.
- A **VPC** and **subnet** configured on Huawei Cloud.


## Step 1: Create a CCE Cluster

1. Log in to the [Huawei Cloud Console](https://console.huaweicloud.com).
2. Navigate to **CCE (Cloud Container Engine)** from the console.
3. Click on **Create Cluster** and select the following options:
   - **Cluster Name**: Choose a name (e.g., `nginx-demo-cluster`).
   - **Version**: Choose a Kubernetes version (e.g., `v1.23+`).
   - **VPC and Subnet**: Use an existing VPC and subnet.
   - **Node Specifications**: Add at least one node to your cluster.

4. Once created, wait for the cluster to enter the **Running** state.

## Step 2: Configure kubectl Access

1. Download the **kubeconfig** file from the **Cluster Access** section in the Huawei Cloud Console.
2. Set up your local environment by copying the `kubeconfig.yaml` file to your machine:
   
   ```bash
   mkdir -p ~/.kube
   mv <path_to_kubeconfig_file> ~/.kube/config
