# Terraform module: AWS ALB Ingress Controller installation

This Terraform module can be used to install the [AWS ALB Ingress Controller](https://github.com/kubernetes-sigs/aws-alb-ingress-controller)
into a Kubernetes cluster.

## Improved integration with Amazon Elastic Kubernetes Service (EKS)

This module can be used to install the ALB Ingress controller into a "vanilla" Kubernetes cluster (which is the default)
or it can be used to integrate tightly with AWS-managed [EKS](https://aws.amazon.com/eks/) clusters which allows the deployed pods to
[use IAM roles for service accounts](https://docs.aws.amazon.com/eks/latest/userguide/enable-iam-roles-for-service-accounts.html).

It is required, that an OpenID connect provider [has already been created](https://www.terraform.io/docs/providers/aws/r/eks_cluster.html#example-iam-role-for-eks-cluster) for your EKS cluster for this feature to work.

Just make sure that you set the variable `k8s_cluster_type` type if running on EKS.

## Examples

### EKS deployment

To deploy the AWS ALB Ingress Controller into an EKS cluster, the following
snippet might be used.

```hclaf
locals {
   # Your AWS EKS Cluster ID goes here.
  "k8s_cluster_name" = "my-k8s-cluster"
}

data "aws_region" "current" {}

data "aws_eks_cluster" "target" {
  name = "local.k8s_cluster_name"
}

data "aws_eks_cluster_auth" "aws_iam_authenticator" {
  name = data.aws_eks_cluster.target.name
}

provider "kubernetes" {
  alias = "eks"
  host                   = data.aws_eks_cluster.target.endpoint
  cluster_ca_certificate = base64decode(data.aws_eks_cluster.target.certificate_authority[0].data)
  token                  = data.aws_eks_cluster_auth.aws_iam_authenticator.token
  load_config_file       = false
}

module "alb_ingress_controller" {
  source  = "iplabs/alb-ingress-controller/kubernetes"
  version = "3.1.0"

  providers = {
    kubernetes = "kubernetes.eks"
  }

  k8s_cluster_type = "eks"
  k8s_namespace    = "kube-system"

  aws_region_name  = data.aws_region.current.name
  k8s_cluster_name = data.aws_eks_cluster.target.name
}
```
