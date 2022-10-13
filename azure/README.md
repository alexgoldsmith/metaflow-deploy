# Metaflow-on-Azure - infrastructure code samples

This folder houses resources to help you get started with running 
Metaflow on Azure.

## terraform/
These sample templates bring up a minimal Metaflow services stack on Azure.

## Steps
```shell
az login
cd azure/terraform
terraform apply -target="module.infra" -var-file=my-vars.tfvars
terraform apply -target="module.services" -var-file=my-vars.tfvars
```