# Please copy paste code below

Amazon Virtual Private Cloud (VPC) is a service that lets you launch AWS resources in a logically isolated virtual network that you define. You have complete control over your virtual networking enviroment, including selection of your own IP address range, creation of subnets, and configuration of route tables and network gateway. By the running below model "vpc1" you will be able build VPC with following configurations:

* 6 subnets: 3 private, 3 public
* Internet gateway 
* NAT gateway
* Elastic IP
* Route tables 

```
module "vpc1" {
  source = "dalerboboev/vpc1/aws"
  region        = "eu-west-2"
  cidr_block    = "10.0.0.0/16"
  public_cidr1  = "10.0.101.0/24"
  public_cidr2  = "10.0.102.0/24"
  public_cidr3  = "10.0.103.0/24"
  private_cidr1 = "10.0.1.0/24"
  private_cidr2 = "10.0.2.0/24"
  private_cidr3 = "10.0.3.0/24"
  tags = {
    Team = "2"
  }
}

```

## Terraform module helm-deploy

This terraform module will help you deploy the helm charts on local.

- [Requirements](#Requirements)

- [Usage](#usage)

- [Variables](#variables)

- [Custom Remote Chart Deployment](#custom-remote-chart-deployment)

- [Custom Local Chart Deployment](#custom-local-chart-deployment)

## Requirements
1. Make sure that you have `kubectl` installed and you have configured your `~/.kube/config` 
2. Make sure that terraform also installed and follows Requirements

  * Kubernetes  >=  v1.14.8

  * Terraform >= 0.11.7


## Usage

1. First you will need to create your own helm chart, follow this [link](https://docs.bitnami.com/kubernetes/how-to/create-your-first-helm-chart/) for detailed instructions. If you would like to quickly create helm chart then run:

```
$ mkdir -p deployments/terraform/charts  
$ cd deployments/terraform

#### for remote chart deployment ####
find the remote charte you want to deploy then add the repo as follow:

$ helm repo add example https://example.github.io/helm-charts
$ helm repo update

#### for local chart deployment ####
$ helm create charts/example

Creating example

$ ls charts/example

Chart.yaml  charts      templates   values.yaml
```

2. Create `module.tf` to call the module on terraform registry, then customize it under data section by stating your custom values as your chart needed.

```
cat <<EOF >module.tf
module "helm_deploy" {
  source                 = "fuchicorp/chart/helm"
  version                = "0.0.10"
  deployment_endpoint    = "example.domain.com"
  deployment_name        = "example-deployment-name"
  deployment_environment = "dev"
  deployment_path        = "example/example"      
  release_version        = "#example chart version"
  remote_chart           = "true"
  enabled                = "true"
  remote_override_values = "${data.template_file.deployment_values.rendered}"
}

data "template_file" "deployment_values" {
  template = <<EOF

#put here your custom values like so:
replicas: 2

EOF
}

EOF
```
