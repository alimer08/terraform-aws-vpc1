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

3. Create a simple output file named `output.tf` and copy and paste the following

    ```
    output "success" {
      value = "${module.helm_deploy.success_output}"
    }
    ```

Follow the file path <br>

```
./module.tf
./output.tf
./charts/
  /example ## Your chart location 
    /Chart.yaml
    /charts
    /templates
    /values.yaml
```


## Variables

For more info, please see the [variables file](https://github.com/fuchicorp/terraform-helm-chart/blob/master/variables.tf).

| Variable                 | Description                                                                                 | Default      | Type            |
| :----------------------- | :------------------------------------------------------------------------------------------ | :----------: | :-------------: |
| `deployment_endpoint`    | Ingress endpoint `example.fuchicorp.com`                                                    | `(Optional)` | `domain/string` |
| `deployment_name`        | The name of the deployment for helm release                                                 | `(Required)` | `string`        |
| `deployment_environment` | Name of the namespace                                                                       | `(Required)` | `string`        |
| `deployment_path`        | path for helm chart on local                                                                | `(Required)` | `string`        |
| `release_version`        | Specify the exact chart version to install.                                                 | `(Optional)` | `string`        |
| `remote_override_values` | Specify the name of the file to override default `values.yaml file`                         | `(Optional)` | `string`        |
| `remote_chart`           | Specify whether to deploy remote_chart to `"true"` or `"false"` default value is `"false"`  | `(Optional)` | `bool`          |
| `enabled`                | Specify if you want to deploy the enabled to `"true"` or `"false"` default value is `"true"`| `(Optional)` | `bool`          |
| `template_custom_vars`   | Template custom veriables you can modify variables by parsting the `template_custom_vars`   | `(Optional)` | `map`           |
| `env_vars`               | Environment veriable for the containers takes map                                           | `(Optional)` | `map`           |
| `timeout`                | If you would like to increase the timeout                                                   | `(Optional)` | `number`        |
| `recreate_pods`          | On update performs pods restart for the resource if applicable.                             | `(Optional)` | `bool`          |       
| `values`                 | Name of the values.yaml file                                                                | `(Optional)` | `string`        |

## Custom Remote Chart Deployment 
In a case of remote chart deployment, you can follow the above instruction and use the `module.tf` as follow

```
cat <<EOF >module.tf

module "helm_deploy_remote" {
  source                 = "fuchicorp/chart/helm"
  version                = "0.0.10"
  deployment_name        = "grafana"
  deployment_environment = "dev"
  deployment_path        = "grafana/grafana"
  enabled                = "true"
  remote_chart           = "true"
  release_version        = "6.22.0"
  remote_override_values = "${data.template_file.deployment_values.rendered}"
}

data "template_file" "deployment_values" {
  template = <<EOF
replicas: 4
EOF
}

EOF
```
    
## Custom Local Chart Deployment 
In a case of local chart deployment, you can follow the above instruction and use the `module.tf` as follow <br>

```
cat <<EOF >module.tf

module "helm_deploy_local" {
  # source = "git::https://github.com/fuchicorp/helm-deploy.git"
  source                 = "fuchicorp/chart/helm"
  version                = "0.0.10"
  deployment_endpoint    = "berbers.fuchicorp.com"
  deployment_name        = "berbers"
  deployment_environment = "dev"
  deployment_path        = "charts/berbers"
  remote_chart           = "false"
  enabled                = "true"
}

EOF
```

## Thanks for using our chart, Enjoy using it! 

```
Developed by FuchiCorp DevOps team 🙂

```
