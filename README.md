# Disabling Storage Access Keys for Azure Storage provider problems

This repo has a small example required to reproduce a problem with the azure storage account terraform provider, where if we disable the storage access keys (which is the recommneded security practice internally), the terraform provider fails to update its state.

## reproduction steps

1. Deploy this terraform code via `terraform init` and then `terraform apply`
2. Disable the storage account access keys in the azure portal (by going to `Configuration` and then disabling `Allow storage account key access
`)
3. Run `terraform plan` and notice that the plan fails with the following error:

```bash

│ Error: retrieving queue properties for Storage Account (Subscription: "xxxxxxxxx"
│ Resource Group Name: "example-resource-group"
│ Storage Account Name: "xxxx"): executing request: unexpected status 403 (403 Key based authentication is not permitted on this storage account.) with KeyBasedAuthenticationNotPermitted: Key based authentication is not permitted on this storage account.
│ RequestId:72bad7cc-c003-0043-6c31-402ab9000000
│ Time:2024-11-26T18:34:40.3675396Z
│ 
│   with azurerm_storage_account.example,
│   on main.tf line 52, in resource "azurerm_storage_account" "example":
│   52: resource "azurerm_storage_account" "example" {
│ 
```

# The solution and workaround

You have to set the flag `storage_use_azuread = true` on the `azurerm` resource provider like so

```terraform
provider "azurerm" {
  storage_use_azuread = true
  # use_msi = true
  features {
    resource_group {
      prevent_deletion_if_contains_resources = false
    }
  }
}
```