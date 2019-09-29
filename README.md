# Installing Anchore Enterprise on EKS

This document will walkthrough the installation of Anchore Enterprise on an Amazon EKS cluster. 

## Prerequisites

- A running Amazon EKS cluster with worker nodes launched. See [EKS Documentation](https://docs.aws.amazon.com/eks/latest/userguide/getting-started.html) for more information on this setup. 
- [Helm](https://helm.sh/) client and server installed and configured with your EKS cluster. 

Once you have an EKS cluster up and running with worker nodes launched, you can verity via the followiing command. 

```

```

## Anchore Helm Chart

Anchore maintains a [Helm chart](https://github.com/helm/charts/tree/master/stable/anchore-engine) to simplify the software installation process. An Enterprise installation of the chart will include the following:

- Anchore Enterprise software
- PostgreSQL (9.6.2)
- Redis (4)

To make the necessary configurations to the Helm chart, create a custom `anchore_values.yaml` file and utilize it during installation. There are many options for configuration with Anchore, for the purposes of this document, I will only change what is required to successfully install Anchore Enterprise. 

**Note:** For this installation, an ALB ingress controller will be used You can read more about Kubernetes Ingress with AWS ALB Ingress Controller [here](https://aws.amazon.com/blogs/opensource/kubernetes-ingress-aws-alb-ingress-controller/)

### Configurations

#### Ingress