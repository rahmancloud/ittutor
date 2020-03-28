# ittutor

## Sesi Perkongsian

### Azure Kubernetes Services 29-03-2020

Antara topik-topik yang disentuh:
* [Azure Portal](http://portal.azure.com/)
* [Azure Cloud Shell](https://docs.microsoft.com/en-us/azure/cloud-shell/overview)
* [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/what-is-azure-cli?view=azure-cli-latest)
* [Terraform azurerm_kubernetes_cluster](https://www.terraform.io/docs/providers/azurerm/r/kubernetes_cluster.html)
* [Helm](https://helm.sh/)
* [Install applications with Helm in Azure Kubernetes Service (AKS)](https://docs.microsoft.com/en-us/azure/aks/kubernetes-helm)
* [Sizes for Linux virtual machines in Azure](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/sizes)

#### Deploy AKS menggunakan Terraform

1. Create resource group:
   ```
   resource "azurerm_resource_group" "example" {
     name     = "example-resources"
     location = "West Europe"
   }
   ```
1. Create service principal:
   ```
   az ad sp create-for-rbac --name ayusmadi-demo --scopes /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/example-resources --role Contributor
   ```
1. Create AKS cluster:
   ```
    resource "azurerm_kubernetes_cluster" "example" {
      name                = "example-aks1"
      location            = azurerm_resource_group.example.location
      resource_group_name = azurerm_resource_group.example.name
      dns_prefix          = "exampleaks1"

      default_node_pool {
        name       = "default"
        node_count = 1
        vm_size    = "Standard_D2_v2"
      }

      service_principal {
        client_id     = "00000000-0000-0000-0000-000000000000"
        client_secret = "00000000000000000000000000000000"
      }

      tags = {
        Environment = "Production"
      }
    }

    output "client_certificate" {
      value = azurerm_kubernetes_cluster.example.kube_config.0.client_certificate
    }

    output "kube_config" {
      value = azurerm_kubernetes_cluster.example.kube_config_raw
    }
   ```
1. Configure kube config:
   ```
   az aks get-credentials --resource-group example-resources --name example-aks1
   ```
1. Deploy any app using helm:
   ```
   helm repo add stable https://kubernetes-charts.storage.googleapis.com/
   helm search repo stable
   helm repo update
   helm install my-nginx-ingress stable/nginx-ingress \
     --set controller.nodeSelector."beta\.kubernetes\.io/os"=linux \
     --set defaultBackend.nodeSelector."beta\.kubernetes\.io/os"=linux
   ```
