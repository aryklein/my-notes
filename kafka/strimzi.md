# Strimzi Operator (Kafka in Kubernetes)

Strimzi is a Kubernetes operator that simplifies the process of running Apache Kafka in Kubernetes. To learn how to use
it, go to the official [documentation](https://strimzi.io/docs/operators/latest/using.html)

## Installing Strimzi in Kubernetes with Helm

Add the Strimzi repository:

```bash
helm repo add strimzi https://strimzi.io/charts/
```

List versions:

```bash
helm search repo strimzi --versions
```
Install:

```bash
helm install strimzi/strimzi-kafka-operator
```

To choose a non-default namespace you can use the `--namespace` option and to choose a specific version use the
`--version` option.

## Automating with Terraform

You can use Terraform to achieve some automation:

```terraform
#
terraform {
  required_version = "~> 1.0.0"

  backend "s3" {
    region               = "us-east-1"
    bucket               = "terraform-state"
    key                  = "terraform.tfstate"
    dynamodb_table       = "kafka-state-lock"
    encrypt              = true
  }

  required_providers {
    helm = {
      source  = "hashicorp/helm"
      version = "2.2.0"
    }
  }
}

#
variable "strimzi_version" {
  type        = string
  default     = "0.24.0"
  description = "Strimzi operator version"
}

variable "config_context" {
  type        = string
  default     = "default"
  description = "Kubernetes config context"
}

#
provider "helm" {
  kubernetes {
    config_path    = "~/.kube/config"
    config_context = var.config_context
  }
}

#
resource "helm_release" "strimzi" {
  name             = "strimzi"
  namespace        = "kafka"
  create_namespace = true
  repository       = "https://strimzi.io/charts"
  chart            = "strimzi-kafka-operator"
  version          = var.strimzi_version
}

#
resource "helm_release" "strimzi" {
  name             = "strimzi"
  namespace        = "kafka"
  create_namespace = true
  repository       = "https://strimzi.io/charts"
  chart            = "strimzi-kafka-operator"
  version          = var.strimzi_version
}
```
