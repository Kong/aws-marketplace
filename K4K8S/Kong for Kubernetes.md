# Kong for Kubernetes

This CloudFormation template based offering is meant for pre-production and production environments where customers want to leverage Kong Community capabilities, running on an EKS Cluster. For example, a Kong Proxy Cache implementation using AWS Elasticache for Redis, externalization of all requests processed by Kong to AWS Elasticsearch Service or integration with AWS Lambda Functions. Launch Kong for Kubernetes in a new EKS Cluster or to an existing one.


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

or you can use the CloudFormation Stack Wizard

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



## Step 3: Deploy Kong for Kubernetes

### Get the EKS Cluster Output Parameters

After creating your EKS Cluster to the CloudFormation results tab to get the following output parameters (each one of them has already its respective value)

1. KubeClusterName=EKS-gGE9I3ePoWUw
2. HelmLambdaArn=arn:aws:lambda:ca-central-1:151743893450:function:eks-k4k8s-EKSStack-167OVSZGLH163-Functi-HelmLambda-1TFUHPM3HIQ2Y
3. KubeConfigPath=s3://eks-k4k8s-eksstack-167ovszglh163-kubeconfigbucket-2fe4uwipoli7/.kube/config.enc
4. KubeManifestLambdaArn=arn:aws:lambda:ca-central-1:151743893450:function:eks-k4k8s-EKSStack-167OVSZGLH16-KubeManifestLambda-1DNBS8T2VF6BT	


![CloudFormation](https://github.com/Kong/aws-marketplace/blob/master/screenshots/EKSClusterParams.png)


### Use the parameters the deploy K4K8S

Again, you can deploy K4K8S with the AWS CLI command:

<pre>
aws cloudformation create-stack --stack-name k4k8s --template-url \
https://k4k8s-cloudformation.s3.amazonaws.com/k4k8s.yaml \
--parameters \
ParameterKey=KubeClusterName,ParameterValue=EKS-gGE9I3ePoWUw \
ParameterKey=HelmLambdaArn,ParameterValue=arn:aws:lambda:ca-central-1:151743893450:function:eks-k4k8s-EKSStack-167OVSZGLH163-Functi-HelmLambda-1TFUHPM3HIQ2Y \
ParameterKey=KubeConfigPath,ParameterValue=s3://eks-k4k8s-eksstack-167ovszglh163-kubeconfigbucket-2fe4uwipoli7/.kube/config.enc \
ParameterKey=KubeManifestLambdaArn,ParameterValue=arn:aws:lambda:ca-central-1:151743893450:function:eks-k4k8s-EKSStack-167OVSZGLH16-KubeManifestLambda-1DNBS8T2VF6BT
</pre>

or you can use the CloudFormation Stack Wizard

![CloudFormation](https://github.com/Kong/aws-marketplace/blob/master/screenshots/CloudFormationStack2.png)




## Step 4: Checking Kong for Kubernetes deployment

<pre>
$ kubectl get pod --all-namespaces
NAMESPACE     NAME                         READY   STATUS    RESTARTS   AGE
kong          kong-kong-58df9bbc9d-gl5nx   2/2     Running   1          2m12s
kube-system   aws-node-tvxr7               1/1     Running   0          17m
kube-system   coredns-b8cff6cf8-2c2nr      1/1     Running   0          23m
kube-system   coredns-b8cff6cf8-lqls6      1/1     Running   0          23m
kube-system   kube-proxy-2gw7z             1/1     Running   0          17m
</pre>

<pre>
$ kubectl get service --all-namespaces
NAMESPACE     NAME              TYPE           CLUSTER-IP      EXTERNAL-IP                                                                  PORT(S)                      AGE
default       kubernetes        ClusterIP      172.20.0.1      <none>                                                                       443/TCP                      25m
kong          kong-kong-proxy   LoadBalancer   172.20.97.128   afd0e739b4e5f11ea9bd40645b942d98-1097734905.ca-central-1.elb.amazonaws.com   80:30880/TCP,443:31916/TCP   4m18s
kube-system   kube-dns          ClusterIP      172.20.0.10     <none>                                                                       53/UDP,53/TCP                25m
</pre>


<pre>
$ http afd0e739b4e5f11ea9bd40645b942d98-1097734905.ca-central-1.elb.amazonaws.com
HTTP/1.1 404 Not Found
Connection: keep-alive
Content-Length: 48
Content-Type: application/json; charset=utf-8
Date: Thu, 13 Feb 2020 12:59:18 GMT
Server: kong/1.4.3
X-Kong-Response-Latency: 0

{
    "message": "no Route matched with those values"
}
</pre>
