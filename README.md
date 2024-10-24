# Creating a CCE Cluster and Deploying NGINX with a Load Balancer on Huawei Cloud

This guide will walk you through creating a CCE (Cloud Container Engine) cluster on Huawei Cloud and deploying an NGINX application using a Load Balancer.

## Prerequisites

Before you start, ensure you have:

- A **Huawei Cloud** account.
- A **VPC** and **subnet** configured on Huawei Cloud.


## Step 1: Create a CCE Cluster

1. Log in to the [Huawei Cloud Console](https://console.huaweicloud.com).
2. Navigate to **CCE (Cloud Container Engine)** from the console.
3. Click on **Create Cluster** and select the following options:
   - **Type**: Choose CCE Standard Cluster
   - **Billing Mode**: Choose Pay-per-use
   - **Cluster Name**: Choose a name (e.g., `nginx-demo-cluster`).
   - **Version**: Choose a Kubernetes version (e.g., `v1.29`).
   - **VPC and Subnet**: Use an existing VPC and subnet.
   - **Node Specifications**: Add at least one node to your cluster.

5. Once created, wait for the cluster to enter the **Running** state.

![image](https://github.com/user-attachments/assets/933392cc-d6d2-449c-a386-2e55cc2a2fed)

## Step 2: Bind EIP to the Cluster
Since we want to access the cluster from outside we need to bind EIP

**PS**: If you do this step after creating nodes etc, you have to download kubectl file again and upload to the shell

![image](https://github.com/user-attachments/assets/1988e57c-7563-4647-8875-e3a484dde1f6)


## Step 3: Create Nodes
Create a node with **EIP**
![image](https://github.com/user-attachments/assets/9439f953-8174-4d67-9d75-44db6e4cbaa2)
![image](https://github.com/user-attachments/assets/0aec6444-a19f-4e57-bb06-bae3d76fbcb9)



## Step 4: Configure kubectl Access

1. Download the **kubeconfig** file by clicking the cluster and on the right panel find congifure kubectl from the **Connection Info**
   
![image](https://github.com/user-attachments/assets/e2093a22-ee2f-43c3-9ad1-a9d8552acba8)   ![image](https://github.com/user-attachments/assets/3be6f093-dad9-4a9a-8c2b-6afeb3b1bb2b)

2. Go and open kubectl shell on the console and and upload **kubeconfig** file to this shell
   
![image](https://github.com/user-attachments/assets/f5ba4449-a994-46fc-a8dd-eed106d5fe24)    ![image](https://github.com/user-attachments/assets/a216952d-5fcb-48b2-a1c4-408298504953)



5. Set up your local environment by copying the `kubeconfig.yaml` file to your machine:
   
   ```bash
   mkdir -p ~/.kube
   mv <path_to_kubeconfig_file> ~/.kube/config
   ```

6. Make sure kubectl is installed:

   ```bash
   kubectl version --client
   ```

7.  Switching Kubernetes Contexts

In Kubernetes, each context in your kubeconfig file represents a specific configuration to access a cluster. You can list all available contexts by running the following command:

   ```bash
   kubectl config get-contexts
   ```

6. Change it to this, to access the cluster via a public IP (EIP)
   
   ```bash
   kubectl config use-context external
   ```
  ![image](https://github.com/user-attachments/assets/1825ce81-b7d9-4d09-8e84-dc9ed32f5813)

7. Try to see the node you created in the previous steps:

   ```bash
   kubectl get nodes
   ```
  ![image](https://github.com/user-attachments/assets/1bfd00dc-a8a2-41be-82c7-4a5c72dff2cf)


## Step 5: Deploy NGINX on the Cluster

You can now deploy an NGINX instance using the following YAML files.

### NGINX Deployment

Save this as `nginx-deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

Apply the deployment:

```bash
kubectl apply -f nginx-deployment.yaml
```

### NGINX Service

Create a `nginx-service.yaml` file:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
  annotations:   
    kubernetes.io/elb.class: union
    kubernetes.io/elb.autocreate: '{
      "type": "public",
      "bandwidth_name": "cce-bandwidth-1551163379627",
      "bandwidth_chargemode": "bandwidth",
      "bandwidth_size": 5,
      "bandwidth_sharetype": "PER",
      "eip_type": "5_bgp"
    }'
    kubernetes.io/elb.enterpriseID: '0'  # Replace '0' if you have an enterprise project ID
    kubernetes.io/elb.lb-algorithm: ROUND_ROBIN
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
  - name: service0
    protocol: TCP
    port: 80
    targetPort: 80
```

Apply the service:

```bash
kubectl apply -f nginx-service.yaml
```

### Check the Service

Once the service is deployed, you can verify it:

```bash
kubectl get services
```
![image](https://github.com/user-attachments/assets/80e9dc76-48ba-4a9c-90d1-7e93e37a4866)

---

# Step 6: Access from outside


Use the external IP of the Load Balancer to access the NGINX web server. You can find the external IP with the command:
   ```bash
   kubectl get services
   ```

Access the service externally: You should be able to access the NGINX service in your web browser by using the external IP:

   ```bash
   http://213.250.136.1:80
   ```

   ![image](https://github.com/user-attachments/assets/93b8d799-e89d-4f5f-9298-3b49ea24e3a8)


You should now be able to access the NGINX welcome page in your browser using the public IP.

![image](https://github.com/user-attachments/assets/b13f7714-64f6-44a3-ba6a-9e5d618a151b)




## To delete:

   ```bash
   kubectl delete service nginx
   ```

   ```bash
   kubectl delete deployment nginx-deployment
   ```
















