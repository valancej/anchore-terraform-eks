# Installing Anchore Enterprise on EKS

This document will walkthrough the installation of Anchore Enterprise on an Amazon EKS cluster. 

## Prerequisites

- A running Amazon EKS cluster with worker nodes launched. See [EKS Documentation](https://docs.aws.amazon.com/eks/latest/userguide/getting-started.html) for more information on this setup. 
- [Helm](https://helm.sh/) client and server installed and configured with your EKS cluster. 

Once you have an EKS cluster up and running with worker nodes launched, you can verity via the followiing command. 

```
$ kubectl get nodes
NAME                             STATUS   ROLES    AGE   VERSION
ip-192-168-2-164.ec2.internal    Ready    <none>   10m   v1.14.6-eks-5047ed
ip-192-168-35-43.ec2.internal    Ready    <none>   10m   v1.14.6-eks-5047ed
ip-192-168-55-228.ec2.internal   Ready    <none>   10m   v1.14.6-eks-5047ed
```

## Anchore Helm Chart

Anchore maintains a [Helm chart](https://github.com/helm/charts/tree/master/stable/anchore-engine) to simplify the software installation process. An Enterprise installation of the chart will include the following:

- Anchore Enterprise software
- PostgreSQL (9.6.2)
- Redis (4)

To make the necessary configurations to the Helm chart, create a custom `anchore_values.yaml` file and utilize it during installation. There are many options for configuration with Anchore, for the purposes of this document, I will only change what is required to successfully install Anchore Enterprise. 

**Note:** For this installation, an ALB ingress controller will be used You can read more about Kubernetes Ingress with AWS ALB Ingress Controller [here](https://aws.amazon.com/blogs/opensource/kubernetes-ingress-aws-alb-ingress-controller/)

### Configurations

Make the following changes below to your `anchore_values.yaml`

#### Ingress

```
ingress:
  enabled: true
  # Use the following paths for GCE/ALB ingress controller
  apiPath: /v1/*
  uiPath: /*
    # apiPath: /v1/
    # uiPath: /
    # Uncomment the following lines to bind on specific hostnames
    # apiHosts:
    #   - anchore-api.example.com
    # uiHosts:
    #   - anchore-ui.example.com
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/scheme: internet-facing
```

#### Anchore Engine API Service

```
# Pod configuration for the anchore engine api service.
anchoreApi:
  replicaCount: 1

  # Set extra environment variables. These will be set on all api containers.
  extraEnv: []
    # - name: foo
    #   value: bar

  # kubernetes service configuration for anchore external API
  service:
    type: NodePort
    port: 8228
    annotations: {}
```

**Note:** Changed the service type to NodePort

#### Anchore Enterprise Global

```
anchoreEnterpriseGlobal:
  enabled: true
```

#### Anchore Enterprise UI

```
anchoreEnterpriseUi:
  # kubernetes service configuration for anchore UI
  service:
    type: NodePort
    port: 80
    annotations: {}
    sessionAffinity: ClientIP
```

**Note:** Changed service type to NodePort.

### AWS EKS Configurations

Download the ALB Ingress manifest and update the `cluster-name` section with the name of your EKS cluster name. 

`wget https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/v1.0.1/docs/examples/alb-ingress-controller.yaml`

```
# Name of your cluster. Used when naming resources created
            # by the ALB Ingress Controller, providing distinction between
            # clusters.
            - --cluster-name=anchore-prod
```