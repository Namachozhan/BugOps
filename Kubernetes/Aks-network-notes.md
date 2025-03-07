# Azure Kubernetes Service (AKS) Networking Guide

## 1. **Azure Kubernetes Service (AKS) Node Requirements**
- AKS requires at least **one node** in the cluster.
- The number of nodes required depends on **workload size, performance needs, and high availability**.
- **Recommended:** At least **two nodes** for redundancy and failover.

## 2. **IP Address Allocation & Subnet Ranges in AKS**
- The number of IP addresses you can request depends on the **Azure Virtual Network (VNet) and Subnet configuration**.
- **Subnet Range Considerations:**
  - Each **node** needs an IP address.
  - Each **pod** also requires an IP address.
  - Use **CIDR notation** to define subnet ranges (e.g., `10.0.0.0/16`).
  - Plan for **sufficient IPs** to accommodate scaling.

## 3. **What is Azure CNI?**
- **Azure Container Networking Interface (CNI)** manages networking for AKS.
- Provides **network connectivity** for **pods and nodes** in Azure.
- **Two CNI Modes in AKS:**
  - **Azure CNI (Node Subnet)** → Assigns pod IPs from the VNet.
  - **Azure CNI Overlay** → Uses a private IP space for pods (does not consume VNet IPs).
- **Best Use Cases:**
  - **Azure CNI Node Subnet** → When pods need direct access to Azure services.
  - **Azure CNI Overlay** → When VNet has limited available IPs.

## 4. **AKS Networking Configuration Options**
### **Container Networking**
- **Azure CNI Overlay** (Recommended for large-scale clusters)
- **Azure CNI Node Subnet** (Best for workloads requiring VNet integration)
- **Bring your own Azure virtual network (VNet)** option available

### **Network Policies**
- **None** → Allows all ingress and egress traffic to pods (least secure).
- **Calico** → Open-source networking policy solution (best for strict security needs).
- **Azure** → Azure-native network policy (simple and easy to use).

## 5. **Best Practices for AKS Networking**
- Choose **Azure CNI Overlay** for better scalability if VNet IPs are limited.
- Use **Calico Network Policies** for high-security workloads.
- Plan **subnet sizes properly** to accommodate scaling.
- Enable **Cilium Dataplane** for advanced security and networking policies.

## 6. **Configuring AKS Networking via Terraform**
```hcl
resource "azurerm_kubernetes_cluster" "aks" {
  name                = "myAKSCluster"
  location            = azurerm_resource_group.rg.location
  resource_group_name = azurerm_resource_group.rg.name
  dns_prefix          = "myaksdns"

  default_node_pool {
    name       = "default"
    node_count = 2
    vm_size    = "Standard_DS2_v2"
  }

  network_profile {
    network_plugin = "azure"
    network_policy = "calico"
  }
}
```

---


