
# Lab 1 - Create EKS cluster and demo application

## Create an Amazon EKS Cluster

> **Note**
>
> Double check that the required tooling is installed before creating a cluster. You need to have the AWS CLI installed and configured and `kubectl` must is installed as well.

Use the following command to create your **EKS** cluster **without** adding ssh keys for access to the worker nodes (ssh access is not required to complete the workshop!):

```bash
eksctl create cluster \
--name dev303-workshop \
--region us-west-2 \
--managed \
--nodes=4
```

**Advanced** Use the following command to create your **EKS** cluster **adding** ssh keys for access to the worker nodes (first generate your ssh keypair if not done already):

```bash
eksctl create cluster \
--name dev303-workshop \
--region us-west-2 \
--managed \
--nodes=4 \
--ssh-access --ssh-public-key=myid_rsa_ssh_key.pub
```

This will create a cluster in the us-west-2 (Oregon) region with 4 x m5.large instances.

> **Note**
> 
> Launching the EKS control plane and worker nodes will take approximately 15 minutes

After **eksctl** successfully created the cluster, check if the worker nodes are online and kubectl is working. Run 
```
kubectl get nodes
```

and you should see a list of nodes:

```bash
$ kubectl get nodes
NAME                                           STATUS   ROLES    AGE     VERSION
ip-192-168-0-159.us-west-2.compute.internal    Ready    <none>   6m1s    v1.16.8-eks-fd1ea7
ip-192-168-12-175.us-west-2.compute.internal   Ready    <none>   6m3s    v1.16.8-eks-fd1ea7
ip-192-168-38-84.us-west-2.compute.internal    Ready    <none>   63s     v1.16.8-eks-fd1ea7
ip-192-168-86-64.us-west-2.compute.internal    Ready    <none>   2m56s   v1.16.8-eks-fd1ea7
```

## Deploy "AnyCompany Shop" microservices application

Follow the steps described next to prepare your environment for the application deployment.

### Deploy application prerequisites

First, deploy the definitions to prepare the environment. This step creates a Kubernetes **namespace** for the microservices and configures the **environment**.

> **Note**
>
> If you are not deploying to the *us-west-2* region you need to update the AWS_REGION variable in the following file `deploy/eks/prep.yaml` to point to the region used.

```
kubectl create -f deploy/eks/prep.yaml
```
> For demo purposes some values are pre-generated. Replace them with your own, base64 encoded, values to improve security.

### Deploy "AnyCompany Shop"

Deploy the eCommerce microservice application to the cluster
```
kubectl create -f deploy/services
```

If everything worked the list of **Pods** in your Amazon EKS cluster should look like this.

```bash
$ kubectl get pods -n microservices-aws
NAME                                       READY   STATUS    RESTARTS   AGE
cartservice-9d5b5db6-2v2t2                 1/1     Running   0          1d
cartservice-9d5b5db6-4hzzj                 1/1     Running   0          1d
catalogservice-74d8fbd66c-7g6k9            1/1     Running   0          1d
catalogservice-74d8fbd66c-wtmtl            1/1     Running   0          1d
frontend-f56974449-695h2                   1/1     Running   0          1d
frontend-f56974449-lnw58                   1/1     Running   0          1d
imageservice-698fb47d49-7gswx              1/1     Running   0          1d
imageservice-698fb47d49-dg4cc              1/1     Running   0          1d
loadgen-6d9c75f66b-sp5cq                   1/1     Running   0          1d
orderservice-7dc7bcbc66-fkfr2              1/1     Running   0          1d
orderservice-7dc7bcbc66-fzgr4              1/1     Running   0          1d
recommenderservice-8bbb5fb66-2425f         1/1     Running   0          1d
recommenderservice-8bbb5fb66-8bcsr         1/1     Running   0          1d
redis-58b5b5d5d4-94fpf                     1/1     Running   0          1d
```

For each microservice, 2 replicas are running. An instance of Redis is online for session and cart storage. The *loadgen* Pod is generating traffic to simplify the next two labs.

### Get Frontend endpoint

Get Frontend SVC Url with the following command 
```
kubectl get svc -o jsonpath='{.items[?(@.metadata.name == "frontend-external")].status.loadBalancer.ingress[0].hostname}' -n microservices-aws
```

Copy the URL into your browser and explore the "AnyCompany Shop". Add a product to your cart and proceed to checkout.

![Storefront](images/storefront.png)

The installation of the "AnyCompany Shop" microservices demo application is now complete.

# Next Step

Continue to [**Lab 2**](lab2.md)
