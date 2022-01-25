# Leaderboard-GKE-QA ![Terraform](https://www.vectorlogo.zone/logos/terraformio/terraformio-icon.svg) ![GCP](https://www.vectorlogo.zone/logos/google_cloud/google_cloud-icon.svg) ![kubernetes](https://www.vectorlogo.zone/logos/kubernetes/kubernetes-icon.svg)

Terraform script to provision GKE (Gcloud Kubernetes Engine). 

## Prerequisites

<details><summary></summary>
</details>

* GCP Account
* Project ID
* Gcloud CLI
* GCP credentials

## Description

This script contains 3 files.
1. `main.tf` for provisioning resources for GKE
2. `variables.tf` for variables that are used in `main.tf`
3. `outputs.tf` for outputs to print out name of the cluster after it provisioned.

<details><summary><strong>main.tf</strong></summary>
In `main.tf` we have our gke module which is terraform own module for kubernetes engine.

```hcl
... 
module "gke" {
  source                 = "terraform-google-modules/kubernetes-engine/google//modules/private-cluster"
  project_id             = var.project_id
  name                   = "${var.cluster_name}-${var.env_name}"
  regional               = false
  region                 = var.region
  zones                  = ["us-central1-c"]
  network                = module.gcp-network.network_name
  subnetwork             = module.gcp-network.subnets_names[0]
  ip_range_pods          = var.ip_range_pods_name
  ip_range_services      = var.ip_range_services_name
  node_pools = [
    {
      name                      = "node-pool"
      machine_type              = "n1-standard-2"
      node_locations            = "us-central1-c"
      min_count                 = var.minnode
      max_count                 = var.maxnode
      disk_size_gb              = var.disksize
    },
  ]
}
```
This module is predefined module of terraform for GKE cluster. 
</details>

<details><summary><strong>variables.tf</strong></summary>

In `variables.tf` we have defined all variables that are used in proviosing our cluster.

```hcl
variable "project_id" {
  description = "The project ID to host the cluster in"
  default = "sonarqube-289802"
}
variable "region" {
  description = "The region to host the cluster in"
  default     = "us-central1"
}
...
variable "disksize" {
  description = "Disk Size in GB"
  default = 10
}
```
where we have variables

* `project_id` for Project ID in which GKE is provisioned.
* `cluster_name` Will be our name of GKE cluster
* `env_name` The environment/namespace for GKE cluster
* `region` Region to host the cluster
* `zones` Cluster zone of nodes
* `network` The name of VPC network created to host the cluster
* `subnetwork` The name of subnetwork created to host the cluster
* `ip_range_pods_name` The name of secondary ip range to use for pods
* `ip_range_services_name` The name of secondary ip range to use for service
* `service-account-id` The ID of our service account in GCP
* `disksize` The size of disk of nodes in GB
* `minnode` The minimum number of nodes
* `maxnode` The maximum number of nodes

</details>





<details><summary><strong>outputs.tf</strong></summary>

In `outputs.tf` we have variables to output when our GKE cluster is provisioned

```hcl
output "cluster_name" {
  description = "Cluster name"
  value       = module.gke.name
}
```

* `cluster_name` Output will be our name of cluster created

</details>

---
## Credentials

1. **Grant permissions to service account _(Skip if already did)_**

```bash
gcloud projects add-iam-policy-binding core-phoenix-330516 --member="serviceAccount:NAME@core-phoenix-330516.iam.gserviceaccount.com" --role="roles/owner"
```
> Change NAME from name of service account

<br>

2. **Generate the key file for credentials**

```bash
gcloud iam service-accounts keys create knolgcp.json --iam-account=NAME@core-phoenix-330516.iam.gserviceaccount.com
```
> Change NAME from name of service account

This will create the credential key `knolgcp.json` in current working directory

3. **To use these credentials export the key**

```bash
export GOOGLE_APPLICATION_CREDENTIALS="/path/to/key.json"
```
## Usage 

1. Initiallize terraform to install provider plugins and configure backend

```bash
terraform init
```

 2. Plan out the terraform script to provision the GKE cluster.

 ```bash
terraform plan -out gke.tfplan
 ```
 3. Apply the out plan to provision the GKE cluster. 

```bash
terrafrom apply gke.tfplan
```
© 2022 GitHub, Inc.
Terms
Privacy
Security
Status
Docs
