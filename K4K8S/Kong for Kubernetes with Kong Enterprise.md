# Kong for Kubernetes with Kong Enterprise

.If you are an Enterprise customer, you have an option of running the [Enterprise version of the Ingress Controller](https://github.com/Kong/aws-marketplace/blob/master/K4K8S/Kong%20for%20Kubernetes%20Enterprise.md), which includes all the Enterprise plugins but does not include Kong Manager or any other Enterprise features. This makes it possible to run the Ingress layer without a database, providing a very low operational and maintance footprint.

However, in some cases, those enterprise features are necessary, and for such use-cases we support another deployment - Kong for Kubernetes with Kong Enterprise.

As seen in the diagram below, this deployment consists of Kong for Kubernetes deployed in Kubernetes, and is hooked up with a database. If there are services running outside Kubernetes, a regular Kong Gateway proxy can be deployed there and connected to the same database. This provides a single pane of visibility of all services that are running in your infrastructure.

![K4K8SwKE](https://github.com/Kong/aws-marketplace/blob/master/screenshots/k4k8s-with-kong-enterprise.png)


#  Installation Process


## Step 1: Key Pair
A [Key Pair](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html) must be configured in the region you plan to deploy your EKS Cluster.

![KeyPair](https://github.com/Kong/aws-marketplace/blob/master/screenshots/KeyPair.png)



## Step 2: Create a AWS EKS Cluster with a CloudFormation stack

You can create your EKS Cluster with the AWS CLI command:

<pre>
aws cloudformation create-stack --stack-name eks-k4k8s --template-url \
https://k4k8s-cloudformation.s3.amazonaws.com/amazon-eks.yaml \
--parameters \
ParameterKey=KeyPairName,ParameterValue=ekskey \
ParameterKey=NumberOfAZs,ParameterValue=2 \
ParameterKey=VPCCIDR,ParameterValue=10.0.0.0/16 \
ParameterKey=PrivateSubnet1CIDR,ParameterValue=10.0.0.0/19 \
ParameterKey=PrivateSubnet2CIDR,ParameterValue=10.0.32.0/19 \
ParameterKey=PublicSubnet1CIDR,ParameterValue=10.0.128.0/20 \
ParameterKey=PublicSubnet2CIDR,ParameterValue=10.0.144.0/20 \
ParameterKey=AvailabilityZones,ParameterValue=ca-central-1a\\,ca-central-1b \
ParameterKey=RemoteAccessCIDR,ParameterValue=0.0.0.0/0 \
ParameterKey=NumberOfNodes,ParameterValue=1 \
--capabilities CAPABILITY_NAMED_IAM
</pre>

or you can use the CloudFormation Stack [Wizard](https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/new?stackName=k4k8s-eks&templateURL=https://k4k8s-cloudformation.s3.amazonaws.com/amazon-eks.yaml)

![CloudFormation](https://github.com/Kong/aws-marketplace/blob/master/screenshots/CloudFormationStack.png)

### Checking your EKS Cluster

This checking assumes you have aws cli and kubectl commands installed locally

### Listing your Clusters
<pre>
$ aws eks list-clusters
{
    "clusters": [
        "EKS-gGE9I3ePoWUw"
    ]
}
</pre>

### Configure kubectl locally with your EKS Cluster info
<pre>
$ aws eks --region ca-central-1 update-kubeconfig --name EKS-gGE9I3ePoWUw
</pre>

### Get your Cluster info
<pre>
$ kubectl cluster-info
Kubernetes master is running at https://06939856D02E23FFFFCB3DA73DA350D3.sk1.ca-central-1.eks.amazonaws.com
CoreDNS is running at https://06939856D02E23FFFFCB3DA73DA350D3.sk1.ca-central-1.eks.amazonaws.com/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
</pre>


### Checking your Pods
<pre>
$ kubectl get pod --all-namespaces
NAMESPACE     NAME                      READY   STATUS    RESTARTS   AGE
kube-system   aws-node-5bz45            1/1     Running   0          9m47s
kube-system   coredns-b8cff6cf8-2682w   1/1     Running   0          13m
kube-system   coredns-b8cff6cf8-nkfx2   1/1     Running   0          13m
kube-system   kube-proxy-4khnp          1/1     Running   0          9m47s
</pre>

### Checking your Services
<pre>
$ kubectl get services --all-namespaces
NAMESPACE     NAME         TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)         AGE
default       kubernetes   ClusterIP   172.20.0.1    <none>        443/TCP         14m
kube-system   kube-dns     ClusterIP   172.20.0.10   <none>        53/UDP,53/TCP   14m
</pre>



## Step 3: Deploy Kong for Kubernetes with Kong Enterprise

### Create a "kong" namespace
$ kubectl create namespace kong

### Create a secret with your license file
$ kubectl create secret generic kong-enterprise-license -n kong --from-file=./license

### Create a secret for the docker registry
$ kubectl create secret -n kong docker-registry kong-enterprise-docker --docker-server=kong-docker-kong-enterprise-k8s.bintray.io --docker-username=\<userid\> --docker-password=\<apikey\> -n kong


### Get the EKS Cluster Output Parameters

After creating your EKS Cluster go to the CloudFormation results tab and get the following output parameters (each one of them has already its respective value)

1. KubeClusterName=EKS-gGE9I3ePoWUw
2. HelmLambdaArn=arn:aws:lambda:ca-central-1:151743893450:function:eks-k4k8s-EKSStack-167OVSZGLH163-Functi-HelmLambda-1TFUHPM3HIQ2Y
3. KubeConfigPath=s3://eks-k4k8s-eksstack-167ovszglh163-kubeconfigbucket-2fe4uwipoli7/.kube/config.enc
4. KubeManifestLambdaArn=arn:aws:lambda:ca-central-1:151743893450:function:eks-k4k8s-EKSStack-167OVSZGLH16-KubeManifestLambda-1DNBS8T2VF6BT	


![CloudFormation](https://github.com/Kong/aws-marketplace/blob/master/screenshots/EKSClusterParams.png)


### Use the parameters the deploy K4K8S with Kong Enterprise

Again, you can deploy K4K8S with Kong Enterprise with the AWS CLI:

<pre>
aws cloudformation create-stack --stack-name k4k8s-eks --template-url \
https://k4k8s-cloudformation.s3.amazonaws.com/k4k8s-enterprise.yaml \
--parameters \
ParameterKey=KubeClusterName,ParameterValue=EKS-gGE9I3ePoWUw \
ParameterKey=HelmLambdaArn,ParameterValue=arn:aws:lambda:ca-central-1:151743893450:function:eks-k4k8s-EKSStack-167OVSZGLH163-Functi-HelmLambda-1TFUHPM3HIQ2Y \
ParameterKey=KubeConfigPath,ParameterValue=s3://eks-k4k8s-eksstack-167ovszglh163-kubeconfigbucket-2fe4uwipoli7/.kube/config.enc \
ParameterKey=KubeManifestLambdaArn,ParameterValue=arn:aws:lambda:ca-central-1:151743893450:function:eks-k4k8s-EKSStack-167OVSZGLH16-KubeManifestLambda-1DNBS8T2VF6BT
</pre>

or you can use the CloudFormation Stack [Wizard](https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/new?stackName=k4k8s-eks&templateURL=https://k4k8s-cloudformation.s3.amazonaws.com/k4k8s-enterprise.yaml)


![CloudFormation](https://github.com/Kong/aws-marketplace/blob/master/screenshots/CloudFormationStack2.png)




## Step 4: Checking Kong for Kubernetes Enterprise deployment

<pre>
$ kubectl get pod --all-namespaces
NAMESPACE     NAME                              READY   STATUS      RESTARTS   AGE
kong          kong-kong-56c9b5f49f-nql9k        1/1     Running     0          9m53s
kong          kong-kong-init-migrations-khnhs   0/1     Completed   0          9m53s
kong          kong-postgresql-0                 1/1     Running     0          9m53s
kube-system   aws-node-qdh9c                    1/1     Running     0          34h
kube-system   coredns-b8cff6cf8-bxgvl           1/1     Running     0          34h
kube-system   coredns-b8cff6cf8-hpf5f           1/1     Running     0          34h
kube-system   kube-proxy-wldb5                  1/1     Running     0          34h
</pre>

<pre>
$ kubectl get service --all-namespaces
NAMESPACE     NAME                       TYPE           CLUSTER-IP       EXTERNAL-IP                                                                 PORT(S)                         AGE
default       kubernetes                 ClusterIP      172.20.0.1       <none>                                                                      443/TCP                         34h
kong          kong-kong-admin            NodePort       172.20.249.243   <none>                                                                      8444:32176/TCP                  10m
kong          kong-kong-manager          NodePort       172.20.20.223    <none>                                                                      8002:32759/TCP,8445:30892/TCP   10m
kong          kong-kong-portal           NodePort       172.20.115.210   <none>                                                                      8003:32073/TCP,8446:31468/TCP   10m
kong          kong-kong-portalapi        NodePort       172.20.251.83    <none>                                                                      8004:31864/TCP,8447:32136/TCP   10m
kong          kong-kong-proxy            LoadBalancer   172.20.238.46    affb57b8a582111eab3ec060c0d5b238-203754153.ca-central-1.elb.amazonaws.com   80:31873/TCP,443:30418/TCP      10m
kong          kong-postgresql            ClusterIP      172.20.20.10     <none>                                                                      5432/TCP                        10m
kong          kong-postgresql-headless   ClusterIP      None             <none>                                                                      5432/TCP                        10m
kube-system   kube-dns                   ClusterIP      172.20.0.10      <none>                                                                      53/UDP,53/TCP                   34h
</pre>


<pre>
$ http affb57b8a582111eab3ec060c0d5b238-203754153.ca-central-1.elb.amazonaws.com
HTTP/1.1 404 Not Found
Connection: keep-alive
Content-Length: 48
Content-Type: application/json; charset=utf-8
Date: Tue, 25 Feb 2020 23:07:06 GMT
Server: kong/1.3.0.2-enterprise-edition

{
    "message": "no Route matched with those values"
}
</pre>



## Kong for Kubernetes official documentation

Browse this [guides](https://github.com/Kong/kubernetes-ingress-controller) to get started configuring Kong for Kubernetes.
