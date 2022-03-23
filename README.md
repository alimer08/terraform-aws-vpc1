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

```sh
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

```yaml
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

```yaml
output "success" {
  value = "${module.helm_deploy.success_output}"
}
```
