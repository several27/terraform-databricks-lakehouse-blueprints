---
page_title: "Provisioning Secure Databricks Workspaces on AWS with Terraform"
---

# How to Deploy a Lakehouse Blueprint using Best Practices and Industry Helper Libraries

This guide uses the following variables. Please all variables here in a terraform.tfvars file (outside the github project) and reference it using `terraform apply -var-file="<location for tfvars file>/terraform.tfvars"`.

- `databricks_account_id`: The ID per Databricks AWS account used for accessing account management APIs. After the AWS E2 account is created, this is available after logging into [https://accounts.cloud.databricks.com](https://accounts.cloud.databricks.com).
- `databricks_account_username`: E2 user account email address
- `databricks_account_password` - E2 account password
- `AWS_PROFILE` - AWS profile used for the AWS CLI and authentication for terraform to AWS APIs
- `AWS_PROFILE` - region in which all AWS resources are deployed
- `relay_vpce_service` - VPC endpoint service for backend relays. This is region-dependent; for example, for us-east-1 the service is available on the Databricks Private Link documentation - `com.amazonaws.vpce.us-east-1.vpce-svc-00018a8c3ff62ffdf`
- `workspace_vpce_service` - VPC endpoint service for workspace communication. This is region-dependent; for example, for us-east-1 the service is available on the Databricks Private Link documentation - `com.amazonaws.vpce.us-east-1.vpce-svc-09143d1e626de2f04`
- `vpce_subnet_cidr` - CIDR for deployment of the VPC endpoint subnets

This guide is provided as-is and you can use this guide as the basis for your custom Terraform module.



## Provider initialization

Initialize [provider with `mws` alias](https://www.terraform.io/language/providers/configuration#alias-multiple-provider-configurations) to set up account-level resources.

```hcl
terraform {
  required_providers {
    databricks = {
      source  = "databricks/databricks"
      version = "0.5.0"
    }
    aws = {
      source = "hashicorp/aws"
      version = "3.49.0"
    }
  }
}

provider "aws" {
  region = var.region
}

// initialize provider in "MWS" mode for provisioning workspace with AWS PrivateLink
provider "databricks" {
  alias    = "mws"
  host     = "https://accounts.cloud.databricks.com"
  username = var.databricks_account_username
  password = var.databricks_account_password
}


```