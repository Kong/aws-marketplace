# Kong for Kubernetes

The Kong for Kubernetes deployment is based on a Terraform template available at https://github.com/terraform-providers/terraform-provider-aws/tree/master/examples/eks-getting-started. 

This template based offer is meant for pre-production and production environments where customers want to leverage Kong for Kubernetes, running on an EKS Cluster.

For more information about the template, please refer to: https://learn.hashicorp.com/tutorials/terraform/eks


#  Installation Process

## Step 1: Install AWS CLI and Terraform
Open a terminal and install AWS CLI as described [here](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli)

The Terraform installation process is documented [here](https://learn.hashicorp.com/tutorials/terraform/install-cli)


## Step 2: Download Terraform Provider Azure Resource Manager repository
![TerraformProvider](https://github.com/Kong/azure-marketplace/blob/master/Kuma/screenshots/github.png)


## Step 3: Login to Azure
Login to Azure with the following command:
<pre>
az login
</pre>
You will be redirected to Azure Portal page. Use your Azure credentials to login. After getting authenticated, you should receive a response like this:

<pre>
You have logged in. Now let us find all the subscriptions to which you have access...
[
  {
    "cloudName": "AzureCloud",
    "homeTenantId": "f177c1d6-zzz-yyy-818a-xxxxxxxd8d",
    "id": "aaaaaa95-bbbd-4fff-b2ba-0ooooooo7948",
    "isDefault": true,
    "managedByTenants": [],
    "name": "Subscription1",
    "state": "Enabled",
    "tenantId": "f177c1d6-zzz-yyy-818a-xxxxxxxd8d",
    "user": {
      "name": "email@youcompany.com",
      "type": "user"
    }
  }
]
</pre>

## Step 4: Create the EKS Cluster
### Initiate Terraform
Open the Zip file you donwloaded and go to the <b>./examples/kubernetes/basic-cluster</b> directory. Run the following command to initiate Terraform:
<pre>
$ terraform init
</pre>

You should see an output like this:
<pre>
Initializing the backend...

Initializing provider plugins...
- Finding latest version of hashicorp/azurerm...
- Installing hashicorp/azurerm v2.32.0...
- Installed hashicorp/azurerm v2.32.0 (signed by HashiCorp)

The following providers do not have any version constraints in configuration,
so the latest version was installed.

To prevent automatic upgrades to new major versions that may contain breaking
changes, we recommend adding version constraints in a required_providers block
in your configuration, with the constraint strings suggested below.

* hashicorp/azurerm: version = "~> 2.32.0"

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
</pre>

### Generate the Terraform plan for the AKS Cluster
The <b>main.tf</b> script creates an AKS Cluster for us. Inside the same directory, generate the Terraform plan with the following command:
<pre>
terraform plan -out k4k8s-eks.tfplan
</pre>

You should see the following output. Notice we have to choose a region where the AKS Cluster will be created and a prefix name for all the resources:
<pre>
$ terraform plan -out k4k8s-eks.tfplan
var.location
  The Azure Region in which all resources in this example should be provisioned

  Enter a value: eastus

var.prefix
  A prefix used for all resources in this example

  Enter a value: kuma

Refreshing Terraform state in-memory prior to plan...
The refreshed state will be used to calculate this plan, but will not be
persisted to local or remote state storage.


------------------------------------------------------------------------

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # azurerm_kubernetes_cluster.example will be created
  + resource "azurerm_kubernetes_cluster" "example" {
      + dns_prefix              = "kuma-k8s"
      + fqdn                    = (known after apply)
      + id                      = (known after apply)
      + kube_admin_config       = (known after apply)
      + kube_admin_config_raw   = (sensitive value)
      + kube_config             = (known after apply)
      + kube_config_raw         = (sensitive value)
      + kubelet_identity        = (known after apply)
      + kubernetes_version      = (known after apply)
      + location                = "eastus"
      + name                    = "kuma-k8s"
      + node_resource_group     = (known after apply)
      + private_cluster_enabled = (known after apply)
      + private_fqdn            = (known after apply)
      + private_link_enabled    = (known after apply)
      + resource_group_name     = "kuma-k8s-resources"
      + sku_tier                = "Free"

      + addon_profile {
          + aci_connector_linux {
              + enabled = false
            }

          + azure_policy {
              + enabled = false
            }

          + http_application_routing {
              + enabled                            = false
              + http_application_routing_zone_name = (known after apply)
            }

          + kube_dashboard {
              + enabled = true
            }

          + oms_agent {
              + enabled            = false
              + oms_agent_identity = (known after apply)
            }
        }

      + auto_scaler_profile {
          + balance_similar_node_groups      = (known after apply)
          + max_graceful_termination_sec     = (known after apply)
          + scale_down_delay_after_add       = (known after apply)
          + scale_down_delay_after_delete    = (known after apply)
          + scale_down_delay_after_failure   = (known after apply)
          + scale_down_unneeded              = (known after apply)
          + scale_down_unready               = (known after apply)
          + scale_down_utilization_threshold = (known after apply)
          + scan_interval                    = (known after apply)
        }

      + default_node_pool {
          + max_pods             = (known after apply)
          + name                 = "default"
          + node_count           = 1
          + orchestrator_version = (known after apply)
          + os_disk_size_gb      = (known after apply)
          + type                 = "VirtualMachineScaleSets"
          + vm_size              = "Standard_DS2_v2"
        }

      + identity {
          + principal_id = (known after apply)
          + tenant_id    = (known after apply)
          + type         = "SystemAssigned"
        }

      + network_profile {
          + dns_service_ip     = (known after apply)
          + docker_bridge_cidr = (known after apply)
          + load_balancer_sku  = (known after apply)
          + network_plugin     = (known after apply)
          + network_policy     = (known after apply)
          + outbound_type      = (known after apply)
          + pod_cidr           = (known after apply)
          + service_cidr       = (known after apply)

          + load_balancer_profile {
              + effective_outbound_ips    = (known after apply)
              + idle_timeout_in_minutes   = (known after apply)
              + managed_outbound_ip_count = (known after apply)
              + outbound_ip_address_ids   = (known after apply)
              + outbound_ip_prefix_ids    = (known after apply)
              + outbound_ports_allocated  = (known after apply)
            }
        }

      + role_based_access_control {
          + enabled = (known after apply)

          + azure_active_directory {
              + admin_group_object_ids = (known after apply)
              + client_app_id          = (known after apply)
              + managed                = (known after apply)
              + server_app_id          = (known after apply)
              + server_app_secret      = (sensitive value)
              + tenant_id              = (known after apply)
            }
        }

      + windows_profile {
          + admin_password = (sensitive value)
          + admin_username = (known after apply)
        }
    }

  # azurerm_resource_group.example will be created
  + resource "azurerm_resource_group" "example" {
      + id       = (known after apply)
      + location = "eastus"
      + name     = "kuma-k8s-resources"
    }

Plan: 2 to add, 0 to change, 0 to destroy.

------------------------------------------------------------------------

This plan was saved to: kuma-aks.tfplan

To perform exactly these actions, run the following command to apply:
    terraform apply "kuma-aks.tfplan"
</pre>

### Create the AKS Cluster
To actually create the AKS Cluster run:
<pre>
terraform apply "kuma-aks.tfplan"
</pre>

### Check the AKS Cluster
<pre>
$ az group list
[
  {
    "id": "/subscriptions/aaaa295-bbbd-ccc7-bddd-eee6952e7948/resourceGroups/kuma-k8s-resources",
    "location": "eastus",
    "managedBy": null,
    "name": "kuma-k8s-resources",
    "properties": {
      "provisioningState": "Succeeded"
    },
    "tags": {},
    "type": "Microsoft.Resources/resourceGroups"
  }
]
</pre>

<pre>
$ az aks list | jq .[].name
"kuma-k8s"
</pre>


### Configure kubectl locally with your AKS Cluster info
<pre>
az aks get-credentials --resource-group kuma-k8s-resources --name kuma-k8s
</pre>

### Get your Cluster info
<pre>
$ kubectl cluster-info
Kubernetes master is running at https://kuma-k8s-e8c02deb.hcp.eastus.azmk8s.io:443
CoreDNS is running at https://kuma-k8s-e8c02deb.hcp.eastus.azmk8s.io:443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
Metrics-server is running at https://kuma-k8s-e8c02deb.hcp.eastus.azmk8s.io:443/api/v1/namespaces/kube-system/services/https:metrics-server:/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
</pre>


### Checking your Pods
<pre>
$ kubectl get pod --all-namespaces
NAMESPACE     NAME                                        READY   STATUS    RESTARTS   AGE
kube-system   coredns-869cb84759-lj6rj                    1/1     Running   0          16m
kube-system   coredns-869cb84759-vkw8f                    1/1     Running   0          17m
kube-system   coredns-autoscaler-5b867494f-zz752          1/1     Running   0          17m
kube-system   dashboard-metrics-scraper-c7b44d7db-rmjhq   1/1     Running   0          17m
kube-system   kube-proxy-f9zfr                            1/1     Running   0          16m
kube-system   kubernetes-dashboard-6788746779-54jt8       1/1     Running   0          17m
kube-system   metrics-server-5f4c878d8-9rvmx              1/1     Running   0          17m
kube-system   tunnelfront-646c8cbcb9-zj6qf                1/1     Running   0          17m
</pre>

### Checking your Services
<pre>
$ kubectl get services --all-namespaces
NAMESPACE     NAME                        TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)         AGE
default       kubernetes                  ClusterIP   10.0.0.1       <none>        443/TCP         18m
kube-system   dashboard-metrics-scraper   ClusterIP   10.0.194.160   <none>        8000/TCP        18m
kube-system   kube-dns                    ClusterIP   10.0.0.10      <none>        53/UDP,53/TCP   18m
kube-system   kubernetes-dashboard        ClusterIP   10.0.122.32    <none>        443/TCP         18m
kube-system   metrics-server              ClusterIP   10.0.169.178   <none>        443/TCP         18m
</pre>



## Step 5: Deploy Kuma Service Mesh
To deploy Kuma Service Mesh use the Terraform Helm provider. Check the official documentation [here](https://registry.terraform.io/providers/hashicorp/helm/latest/docs).

In a new directory named <b>Kuma</b>, create a new <b>main.tf</b> like this:
<pre>
resource "helm_release" "example" {
  name = "kuma"
  repository = "https://kumahq.github.io/charts"
  chart = "kuma"
  version = "0.3.1"
}
</pre>

Apply the Terraform template with the following commands:
<pre>
terraform init
terraform plan -out kuma.tfplan
terraform apply "kuma.tfplan"
</pre>



## Step 6: Checking Kuma Service Mesh deployment

<pre>
$ kubectl get pod --all-namespaces
NAMESPACE     NAME                                        READY   STATUS    RESTARTS   AGE
default       kuma-control-plane-7994db7688-psvnp         1/1     Running   0          21s
kube-system   coredns-869cb84759-lj6rj                    1/1     Running   0          44m
kube-system   coredns-869cb84759-vkw8f                    1/1     Running   0          45m
kube-system   coredns-autoscaler-5b867494f-zz752          1/1     Running   0          45m
kube-system   dashboard-metrics-scraper-c7b44d7db-rmjhq   1/1     Running   0          45m
kube-system   kube-proxy-f9zfr                            1/1     Running   0          45m
kube-system   kubernetes-dashboard-6788746779-54jt8       1/1     Running   0          45m
kube-system   metrics-server-5f4c878d8-9rvmx              1/1     Running   0          45m
kube-system   tunnelfront-646c8cbcb9-zj6qf                1/1     Running   0          45m
</pre>

<pre>
$ kubectl get service --all-namespaces
NAMESPACE     NAME                        TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)                                                                  AGE
default       kubernetes                  ClusterIP   10.0.0.1       <none>        443/TCP                                                                  46m
default       kuma-control-plane          ClusterIP   10.0.92.236    <none>        5681/TCP,443/TCP,5676/TCP,5677/TCP,5678/TCP,5679/TCP,5682/TCP,5653/UDP   63s
kube-system   dashboard-metrics-scraper   ClusterIP   10.0.194.160   <none>        8000/TCP                                                                 46m
kube-system   kube-dns                    ClusterIP   10.0.0.10      <none>        53/UDP,53/TCP                                                            46m
kube-system   kubernetes-dashboard        ClusterIP   10.0.122.32    <none>        443/TCP                                                                  46m
kube-system   metrics-server              ClusterIP   10.0.169.178   <none>        443/TCP                                                                  46m
</pre>

Check Kuma GUI also. On on terminal expose the Control Plane port

<pre>
$ kubectl port-forward service/kuma-control-plane 5681
Forwarding from 127.0.0.1:5681 -> 5681
Forwarding from [::1]:5681 -> 5681
</pre>

Redirect your browser to http://localhost:5681/gui and click on <b>Skip to Dashboard</b>
![KumaGUI](https://github.com/Kong/azure-marketplace/blob/master/Kuma/screenshots/gui.png)


## Kuma official documentation

Browse these [guides](https://kuma.io/docs) to get started configuring Kuma.
