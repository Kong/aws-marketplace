# Kong Mesh

This CloudFormation template based offer is meant for pre-production and production environments where customers want to leverage Kong Mesh capabilities, running on an EKS Cluster. Launch Kong Mesh in a new EKS Cluster or to an existing one.


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
ParameterKey=KeyPairName,ParameterValue=ekskey \
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

or you can use the CloudFormation Stack [Wizard](https://console.aws.amazon.com/cloudformation/home?region=eu-central-1#/stacks/new?stackName=eks-kuma&templateURL=https://kuma-cloudformation.s3.amazonaws.com/amazon-eks.yaml)

![CloudFormation](https://github.com/Kong/aws-marketplace/blob/master/Kuma/screenshots/CF-step1.png)

Click on <b>Next</b>

![CloudFormation](https://github.com/Kong/aws-marketplace/blob/master/Kuma/screenshots/CF-step2.png)

Click on <b>Next</b> and on <b>Next</b> again to go the <b>Review Page</b>

![CloudFormation](https://github.com/Kong/aws-marketplace/blob/master/Kuma/screenshots/CF-step3.png)

On the bottom of the page click on both check boxes: <b>I acknowledge that AWS CloudFormation might create IAM resources with custom names.</b> and <b>I acknowledge that AWS CloudFormation might require the following capability: CAPABILITY_AUTO_EXPAND</b>.

Click on "Create stack"



### Checking your EKS Cluster

This checking assumes you have aws cli and kubectl commands installed locally

### Listing your Clusters
<pre>
$ aws eks list-clusters
{
    "clusters": [
        "eks-kuma"
    ]
}
</pre>

### Configure kubectl locally with your EKS Cluster info
<pre>
$ aws eks --region eu-central-1 update-kubeconfig --name eks-kuma
</pre>

### Get your Cluster info
<pre>
$ kubectl cluster-info
Kubernetes master is running at https://43C4C665F3F1283D711B41CDFE53FA2D.gr7.eu-central-1.eks.amazonaws.com
CoreDNS is running at https://43C4C665F3F1283D711B41CDFE53FA2D.gr7.eu-central-1.eks.amazonaws.com/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
</pre>


### Checking your Pods
<pre>
$ kubectl get pod --all-namespaces
NAMESPACE     NAME                      READY   STATUS    RESTARTS   AGE
kube-system   aws-node-qg8gg            1/1     Running   0          111m
kube-system   coredns-5fdf64ff8-9pk9n   1/1     Running   0          125m
kube-system   coredns-5fdf64ff8-tbbz9   1/1     Running   0          125m
kube-system   kube-proxy-54vxr          1/1     Running   0          111m
</pre>

### Checking your Services
<pre>
$ kubectl get services --all-namespaces
NAMESPACE     NAME         TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)         AGE
default       kubernetes   ClusterIP   172.20.0.1    <none>        443/TCP         125m
kube-system   kube-dns     ClusterIP   172.20.0.10   <none>        53/UDP,53/TCP   125m
</pre>



## Step 3: Deploy Kong Mesh

### Deploy Kong Mesh with AWS CLI

<pre>
aws cloudformation create-stack --stack-name kuma --template-url \
https://kuma-cloudformation.s3.amazonaws.com/kuma.yaml \
--parameters \
ParameterKey=Cluster,ParameterValue=eks-kuma
</pre>

or you can use the [CloudFormation Stack](https://console.aws.amazon.com/cloudformation/home?region=eu-central-1#/stacks/new?stackName=kuma&templateURL=https://kuma-cloudformation.s3.amazonaws.com/kuma.yaml)


![CloudFormation](https://github.com/Kong/aws-marketplace/blob/master/Kuma/screenshots/CF-step4.png)



## Step 4: Checking Kuma Service Mesh deployment

<pre>
$ kubectl get pod --all-namespaces
NAMESPACE     NAME                                  READY   STATUS    RESTARTS   AGE
default       kuma-control-plane-7db4999799-hzmrc   1/1     Running   0          20s
kube-system   aws-node-qg8gg                        1/1     Running   0          126m
kube-system   coredns-5fdf64ff8-9pk9n               1/1     Running   0          139m
kube-system   coredns-5fdf64ff8-tbbz9               1/1     Running   0          139m
kube-system   kube-proxy-54vxr                      1/1     Running   0          126m
</pre>

<pre>
$ kubectl get service --all-namespaces
NAMESPACE     NAME                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                                                                  AGE
default       kubernetes           ClusterIP   172.20.0.1       <none>        443/TCP                                                                  140m
default       kuma-control-plane   ClusterIP   172.20.109.231   <none>        5681/TCP,443/TCP,5676/TCP,5677/TCP,5678/TCP,5679/TCP,5682/TCP,5653/UDP   35s
kube-system   kube-dns             ClusterIP   172.20.0.10      <none>        53/UDP,53/TCP                                                            140m
</pre>

Check Kuma GUI also. On on terminal expose the Control Plane port

<pre>
$ kubectl port-forward service/kuma-control-plane 5681:5681
Forwarding from 127.0.0.1:5681 -> 5681
Forwarding from [::1]:5681 -> 5681
</pre>

Redirect your browser to http://localhost:5681/gui and click on <b>Skip to Dashboard</b>
![KumaGUI](https://github.com/Kong/aws-marketplace/blob/master/Kuma/screenshots/GUI.png)


## Kuma official documentation

Browse these [guides](https://kuma.io/docs) to get started configuring Kuma.
