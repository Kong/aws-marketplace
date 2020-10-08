# Kuma Service Mesh

This CloudFormation template based offer is meant for pre-production and production environments where customers want to leverage Kuma Service Mesh capabilities, running on an EKS Cluster. Launch Kong for Kubernetes Enterprise in a new EKS Cluster or to an existing one.


#  Installation Process


## Step 1: Key Pair
A [Key Pair](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html) must be configured in the region you plan to deploy your EKS Cluster.

![KeyPair](https://github.com/Kong/aws-marketplace/blob/master/screenshots/KeyPair.png)



## Step 2: Create a AWS EKS Cluster with a CloudFormation stack

You can create your EKS Cluster with the AWS CLI command:

<pre>
aws cloudformation create-stack --stack-name eks-kuma --template-url \
https://kuma-cloudformation.s3.amazonaws.com/amazon-eks.yaml \
--parameters \
ParameterKey=KeyPairName,ParameterValue=acqua-frankfurt \
ParameterKey=NumberOfAZs,ParameterValue=3 \
ParameterKey=VPCCIDR,ParameterValue=10.0.0.0/16 \
ParameterKey=PrivateSubnet1CIDR,ParameterValue=10.0.0.0/19 \
ParameterKey=PrivateSubnet2CIDR,ParameterValue=10.0.32.0/19 \
ParameterKey=PrivateSubnet3CIDR,ParameterValue=10.0.64.0/19 \
ParameterKey=PublicSubnet1CIDR,ParameterValue=10.0.128.0/20 \
ParameterKey=PublicSubnet2CIDR,ParameterValue=10.0.144.0/20 \
ParameterKey=PublicSubnet3CIDR,ParameterValue=10.0.160.0/20 \
ParameterKey=AvailabilityZones,ParameterValue=eu-central-1a\\,eu-central-1b\\,eu-central-1c \
ParameterKey=RemoteAccessCIDR,ParameterValue=0.0.0.0/0 \
ParameterKey=ProvisionBastionHost,ParameterValue=Disabled \
ParameterKey=NumberOfNodes,ParameterValue=1 \
ParameterKey=EKSClusterName,ParameterValue=eks-kuma \
ParameterKey=EKSPublicAccessEndpoint,ParameterValue=Enabled \
ParameterKey=ALBIngressController,ParameterValue=Disabled \
ParameterKey=EKSPublicAccessEndpoint,ParameterValue=Enabled \
--capabilities CAPABILITY_NAMED_IAM CAPABILITY_AUTO_EXPAND
</pre>

or you can use the CloudFormation Stack [Wizard](https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/new?stackName=kuma-eks&templateURL=https://kuma-cloudformation.s3.amazonaws.com/amazon-eks.yaml)

![CloudFormation](https://github.com/Kong/aws-marketplace/blob/master/Kuma/screenshots/CloudFormation-step1.png)

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



## Step 3: Deploy Kong for Kubernetes Enterprise

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


### Use the parameters the deploy K4K8S Enterprise

Again, you can deploy K4K8S with the AWS CLI:

<pre>
aws cloudformation create-stack --stack-name k4k8s --template-url \
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
NAMESPACE     NAME                         READY   STATUS    RESTARTS   AGE
kong          kong-kong-556d9c4b4d-cbw7r   2/2     Running   1          100s
kube-system   aws-node-qdh9c               1/1     Running   0          34h
kube-system   coredns-b8cff6cf8-bxgvl      1/1     Running   0          34h
kube-system   coredns-b8cff6cf8-hpf5f      1/1     Running   0          34h
kube-system   kube-proxy-wldb5             1/1     Running   0          34h
</pre>

<pre>
$ kubectl get service --all-namespaces
NAMESPACE     NAME                  TYPE           CLUSTER-IP       EXTERNAL-IP                                                                 PORT(S)                         AGE
default       kubernetes            ClusterIP      172.20.0.1       <none>                                                                      443/TCP                         34h
kong          kong-kong-manager     NodePort       172.20.55.17     <none>                                                                      8002:31652/TCP,8445:32368/TCP   117s
kong          kong-kong-portal      NodePort       172.20.235.210   <none>                                                                      8003:32260/TCP,8446:32000/TCP   117s
kong          kong-kong-portalapi   NodePort       172.20.84.213    <none>                                                                      8004:31032/TCP,8447:30923/TCP   117s
kong          kong-kong-proxy       LoadBalancer   172.20.184.88    a8ecc6761581f11eaafac02f0c23a054-836684746.ca-central-1.elb.amazonaws.com   80:30964/TCP,443:30719/TCP      117s
kube-system   kube-dns              ClusterIP      172.20.0.10      <none>                                                                      53/UDP,53/TCP                   34h
</pre>


<pre>
$ http a8ecc6761581f11eaafac02f0c23a054-836684746.ca-central-1.elb.amazonaws.com
HTTP/1.1 404 Not Found
Connection: keep-alive
Content-Length: 48
Content-Type: application/json; charset=utf-8
Date: Tue, 25 Feb 2020 22:40:56 GMT
Server: kong/1.4.2.0-enterprise-k8s
X-Kong-Response-Latency: 1

{
    "message": "no Route matched with those values"
}
</pre>

<pre>
$ kubectl logs kong-kong-556d9c4b4d-cbw7r -n kong ingress-controller
-------------------------------------------------------------------------------
Kong Ingress controller
  Release:    0.7.1
  Build:      1527700
  Repository: git@github.com:kong/kubernetes-ingress-controller.git
  Go:         go1.13.1
-------------------------------------------------------------------------------

I0225 22:38:37.692355       1 main.go:407] Creating API client for https://172.20.0.1:443
I0225 22:38:37.701145       1 main.go:451] Running in Kubernetes Cluster version v1.14+ (v1.14.9-eks-502bfb) - git (clean) commit 502bfb383169b124d87848f89e17a04b9fc1f6f0 - platform linux/amd64
I0225 22:38:37.915516       1 main.go:187] kong version: 1.4.2-0-enterprise-k8s
I0225 22:38:37.915541       1 main.go:196] Kong datastore: off
I0225 22:38:38.037913       1 controller.go:224] starting Ingress controller
I0225 22:38:38.045261       1 status.go:201] new leader elected: kong-kong-556d9c4b4d-cbw7r
I0225 22:38:38.105963       1 kong.go:66] successfully synced configuration to Kong
I0225 22:38:41.371978       1 kong.go:57] no configuration change, skipping sync to Kong
I0225 22:38:44.774218       1 kong.go:57] no configuration change, skipping sync to Kong
</pre>


## Kong for Kubernetes official documentation

Browse this [guides](https://github.com/Kong/kubernetes-ingress-controller) to get started configuring Kong for Kubernetes.
