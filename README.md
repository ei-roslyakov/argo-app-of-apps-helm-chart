# Helm Chart for Argo CD Applications

This Helm chart allows you to deploy multiple applications using Argo CD. It supports various Kubernetes resources, including AWS Load Balancer Controller, Nginx Ingress Controller, Cluster Autoscaler, and others, providing a flexible and customizable deployment process.

## Table of Contents

- [Overview](#overview)
- [Installation](#installation)
- [Usage](#usage)
- [Configuration](#configuration)
  - [Global Configuration](#global-configuration)
  - [Application-Specific Configuration](#application-specific-configuration)
- [Examples](#examples)
  - [Helm Installation Example](#helm-installation-example)
  - [Argo CD Installation Example](#argo-cd-installation-example)
- [Contributing](#contributing)
- [License](#license)

## Overview

This Helm chart is designed to deploy multiple applications managed by Argo CD in a Kubernetes cluster. It supports various controllers and drivers, such as AWS Load Balancer Controller, Nginx Ingress Controller, Cluster Autoscaler, AWS EFS CSI Driver, and AWS Node Termination Handler.

## Installation

To install the Helm chart, add this Helm repository and use the `helm install` command:

```bash
helm repo add my-helm-repo https://your-helm-repo-url
helm repo update
helm install my-release my-helm-repo/app-of-apps -f values.yaml

Replace my-release with your desired release name and -f `values.yaml` with your custom values file.

Usage
You can enable or disable specific applications by setting the appropriate enabled flag in the values.yaml file. Each application can be configured independently, allowing for flexible deployment management.

Enabling Applications
To enable an application, set its enabled flag to true in the values.yaml file. For example, to enable the AWS Load Balancer Controller:

```yaml
awsLoadBalancerController:
  enabled: true
  name: "aws-load-balancer-controller"
  namespace: yahoo-use2-argocd
  project: yahoo-use2-infrastructure
  source:
    repoURL: "https://aws.github.io/eks-charts"
    targetRevision: 1.6.2
    chart: aws-load-balancer-controller
    helm:
      values: |
        clusterName: c8-eks-cluster-yahoo-use2
        createIngressClassResource: true
        region: us-east-2
        vpcId: vpc-0ac15f3b191287be0
```

Configuration
Global Configuration
The following global parameters can be configured in the values.yaml file:

Parameter	Description	Default
namePrefix	Prefix for application names	myapp-
nameSuffix	Suffix for application names	-use2


Application-Specific Configuration
Each application has its own set of configurable parameters in the values.yaml file. Below are the common parameters:

enabled: Enable or disable the application deployment.
name: The name of the application.
namespace: The Kubernetes namespace where the application will be deployed.
project: The Argo CD project name.
destination: The destination cluster and namespace for the application.
source: The source Helm repository URL, chart, and version.
syncPolicy: Synchronization policies for Argo CD, including automated sync options.

Example `values.yaml`
Here is an example configuration for multiple applications:

```yaml
namePrefix: "myapp-"
nameSuffix: "-use2"

metricsServer:
  enabled: false
  name: "metrics-server"
  namespace: 
  project: 
  destination:
    server: "https://kubernetes.default.svc"
    namespace: kube-system
  source:
    repoURL: "https://kubernetes-sigs.github.io/metrics-server/"
    targetRevision: "3.11.0"
    chart: metrics-server

clusterAutoscaler:
  enabled: false
  name: "cluster-autoscaler"
  namespace: 
  project: 
  destination:
    server: "https://kubernetes.default.svc"
    namespace: kube-system
  source:
    repoURL: "https://kubernetes.github.io/autoscaler"
    targetRevision: 9.37.0
    chart: cluster-autoscaler
    helm:
        values: |
            autoDiscovery:
            clusterName: "c8-eks-cluster-yahoo-use2"

awsLoadBalancerController:
    enabled: true
    name: "aws-load-balancer-controller"
    namespace: yahoo-use2-argocd
    project: yahoo-use2-infrastructure
    source:
    repoURL: "https://aws.github.io/eks-charts"
    targetRevision: 1.6.2
    chart: aws-load-balancer-controller
    helm:
        values: |
        clusterName: c8-eks-cluster-yahoo-use2

        ingresses:
            - name: c8-yahoo-use2-external
            namespace: yahoo-use2-ingress-nginx
            annotations:
                alb.ingress.kubernetes.io/scheme: internet-facing
                alb.ingress.kubernetes.io/target-type: ip
                alb.ingress.kubernetes.io/group.name: c8-yahoo-use2-external
                alb.ingress.kubernetes.io/load-balancer-name: c8-yahoo-use2-external
                alb.ingress.kubernetes.io/tags: business_unit=Yahoo,team=DevOps,owner=configure8,ManagedBy=Argo,application=c8
                alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:us-east-2:120569638418:certificate/19009c7e-eab5-4e18-8aa8-5c367a92276c
                alb.ingress.kubernetes.io/listen-ports: '[{"HTTP": 80}, {"HTTPS":443}]'
                alb.ingress.kubernetes.io/actions.ssl-redirect: '{"Type": "redirect", "RedirectConfig": { "Protocol": "HTTPS", "Port": "443", "StatusCode": "HTTP_301"}}'
                alb.ingress.kubernetes.io/healthcheck-path: /healthz
                alb.ingress.kubernetes.io/load-balancer-attributes: deletion_protection.enabled=false
                alb.ingress.kubernetes.io/ssl-policy: ELBSecurityPolicy-TLS13-1-2-2021-06
                
                
```