# terraform-google-tfegke

Terraform module to deploy [Terraform Enterprise](https://www.hashicorp.com/products/terraform/enterprise) on Google Kubernetes Engine (GKE) with Google Cloud SQL, Memorystore, Google Cloud Storage, and ACME-managed certificates.

<!-- BEGIN_TF_DOCS -->

## Requirements

| Name       | Version |
| ---------- | ------- |
| terraform  | >= 1.3  |
| google     | 6.3.0   |
| kubernetes | 2.32.0  |
| helm       | 2.15.0  |
| tls        | 4.0.6   |
| random     | 3.6.3   |
| null       | 3.2.3   |
| acme       | 2.26.0  |
| time       | 0.12.1  |

## Providers

| Name       | Version |
| ---------- | ------- |
| google     | 6.3.0   |
| kubernetes | 2.32.0  |
| helm       | 2.15.0  |
| tls        | 4.0.6   |
| random     | 3.6.3   |
| null       | 3.2.3   |
| acme       | 2.26.0  |
| time       | 0.12.1  |

## Inputs

| Name                     | Description                                  | Type       | Default                 |    Required    |
| ------------------------ | -------------------------------------------- | ---------- | ----------------------- | :-------------: |
| `project_id`           | Your GCP project ID                          | `string` | n/a                     |       yes       |
| `region`               | GCP region for resources                     | `string` | `"europe-west1"`      |       no       |
| `location`             | Location for global resources                | `string` | `"global"`            |       no       |
| `vpc_name`             | Name of the VPC to create or use             | `string` | n/a                     |       yes       |
| `create_network`       | Whether to create a new VPC and subnets      | `bool`   | `true`                |       no       |
| `vpc_reference`        | ID or self_link of an existing VPC           | `string` | `""`                  |       no       |
| `subnet1-region`       | CIDR for main subnet                         | `string` | `"10.0.1.0/24"`       |       no       |
| `subnet2-region`       | CIDR for proxy-only subnet                   | `string` | `"10.0.2.0/24"`       |       no       |
| `subnet_reference`     | ID or self_link of an existing subnet        | `string` | `""`                  |       no       |
| `gke_autopilot_enable` | Enable GKE Autopilot mode                    | `bool`   | `false`               |       no       |
| `machine_type`         | GKE node machine type                        | `string` | `"e2-standard-8"`     |       no       |
| `node_count`           | Number of nodes in the GKE cluster           | `number` | `1`                   |       no       |
| `with_node_pool`       | Use node pools (not for Autopilot)           | `bool`   | `false`               |       no       |
| `storage_location`     | GCS bucket location                          | `string` | `"EU"`                |       no       |
| `dns_zone_name_ext`    | Name of the pre-created external DNS zone    | `string` | n/a                     |       yes       |
| `email`                | Email for ACME certificate registration      | `string` | n/a                     |       yes       |
| `acme_prod`            | Use ACME production endpoint                 | `bool`   | `false`               |       no       |
| `tfe_license`          | Terraform Enterprise license                 | `string` | `"empty"`             | yes (sensitive) |
| `k8s_namespace`        | Kubernetes namespace for TFE                 | `string` | `"tfe"`               |       no       |
| `tfe_helm_release`     | Helm chart version for TFE                   | `string` | `"1.3.2"`             |       no       |
| `expose`               | Load balancer type: "Internal" or "External" | `string` | `"External"`          |       no       |
| `db_username`          | PostgreSQL username                          | `string` | n/a                     | yes (sensitive) |
| `db_password`          | PostgreSQL password                          | `string` | n/a                     | yes (sensitive) |
| `instance_name`        | PostgreSQL instance name                     | `string` | `"postgres-instance"` |       no       |
| `tfe_version`          | Terraform Enterprise version                 | `string` | `"v202409-3"`         |       no       |

## Outputs

| Name                                      | Description                                            |
| ----------------------------------------- | ------------------------------------------------------ |
| `project_id`                            | GCloud Project ID                                      |
| `kubernetes_cluster_name`               | GKE Cluster Name                                       |
| `kubernetes_cluster_host`               | GKE Cluster Host                                       |
| `configure_kubectl`                     | gcloud command to configure kubeconfig                 |
| `test`                                  | TFE domain                                             |
| `helm`                                  | Helm values for TFE deployment (sensitive)             |
| `retrieve_initial_admin_creation_token` | URL to retrieve the initial admin creation token       |
| `create_initial_admin_user`             | URL to create the initial admin user                   |
| `remove_database_before_destroy`        | Command to remove the Postgres instance before destroy |
| `remove_peering_before_destroy`         | Command to remove VPC peering before destroy           |

## Usage

```hcl
module "tfe_gke" {
  source                = "./"
  project_id            = "your-gcp-project"
  region                = "europe-west1"
  vpc_name              = "demo"
  dns_zone_name_ext     = "your-dns-zone"
  email                 = "your@email.com"
  db_username           = "admin-user"
  db_password           = "your-secure-password"
  tfe_license           = "your-tfe-license"
  acme_prod             = true
  expose                = "External"
  tfe_version           = "v202505-1"
  # ...other variables as needed
}
```

## Destroying

Before destroying the infrastructure, follow the steps in [`delete_db_peering.ipynb`](delete_db_peering.ipynb) to safely remove the database and VPC peering.

## Files

- [variables.tf](variables.tf)
- [variables.tfvars](variables.tfvars)
- [network.tf](network.tf)
- [gke.tf](gke.tf)
- [postgreSQL.tf](postgreSQL.tf)
- [redis.tf](redis.tf)
- [gcs.tf](gcs.tf)
- [cert.tf](cert.tf)
- [iam.tf](iam.tf)
- [tfe-helm.tf](tfe-helm.tf)
- [output.tf](output.tf)
- [delete_db_peering.ipynb](delete_db_peering.ipynb)
- [templates/tfe-values.yaml.tpl](templates/tfe-values.yaml.tpl)

## Notes

- This module is intended for demonstration and testing purposes. For production, review security and permissions carefully.
- You must pre-create the external DNS zone in your GCP project.

<!-- END_TF_DOCS -->
