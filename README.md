# Helm Chart for Argo CD Applications

This Helm chart allows you to deploy multiple applications using Argo CD. It supports various Kubernetes resources, including AWS Load Balancer Controller, Nginx Ingress Controller, Cluster Autoscaler, and others, providing a flexible and customizable deployment process.

| Application                    | Description                                                                                     |
|---------------------------------|-------------------------------------------------------------------------------------------------|
| `aws-efs-csi-driver`            | AWS EFS CSI Driver for mounting EFS volumes in Kubernetes pods.                                 |
| `aws-load-balancer-controller`  | AWS Load Balancer Controller for managing AWS ELB/NLB resources.                                |
| `aws-node-termination-handler`  | AWS Node Termination Handler for handling EC2 instance terminations gracefully.                 |
| `cluster-autoscaler`            | Kubernetes Cluster Autoscaler for automatic cluster size adjustment.                            |
| `external-secrets`              | Sync secrets from external management systems (AWS Secrets Manager, etc.) into Kubernetes.       |
| `loki-stack`                    | Prometheus-based logging solution with Grafana for log aggregation and monitoring.              |
| `metric-server`                 | Metrics Server for Kubernetes, provides resource metrics to Kubernetes components.              |
| `nginx-ingress-controller`      | Nginx Ingress Controller for managing external access to Kubernetes services.                   |
| `promtail`                      | Promtail is a log shipping agent for Loki.                                                      |
| `rabbitmq-cluster-operator`     | Manages RabbitMQ clusters on Kubernetes.                                                        |
| `redis`                         | Redis deployment for in-memory caching and data storage in Kubernetes.                         |
| `kube-prometheus-stack`         | Monitoring stack built with Prometheus and Grafana.                                             |


## Overview

This Helm chart is designed to deploy multiple applications managed by Argo CD in a Kubernetes cluster. It supports various controllers and drivers, such as AWS Load Balancer Controller, Nginx Ingress Controller, Cluster Autoscaler, AWS EFS CSI Driver, and AWS Node Termination Handler.

## Installation

To install the Helm chart, add this Helm repository and use the `helm install` command:

```bash
helm install my-release my-helm-repo/app-of-apps -f values.yaml
```

Replace my-release with your desired release name and -f `values.yaml` with your custom values file.

Usage
You can enable or disable specific applications by setting the appropriate enabled flag in the values.yaml file. Each application can be configured independently, allowing for flexible deployment management.

Enabling Applications
To enable an application, set its enabled flag to true in the values.yaml file. For example, to enable the AWS Load Balancer Controller:

```yaml
awsLoadBalancerController:
  enabled: true

nginxIngressController:
  enabled: false
```

## Configuration

Global Configuration
The following global parameters can be configured in the values.yaml file:

| Parameter   | Description                     | Default |
|-------------|---------------------------------|---------|
| namePrefix  | Prefix for application names     | myapp-  |
| nameSuffix  | Suffix for application names     | -use2   |

### Application-Specific Configuration
Each application has its own set of configurable parameters in the `values.yaml` file. Below are the common parameters:

* `enabled`: Enable or disable the application deployment.
* `name`: The name of the application.
* `namespace`: The Kubernetes namespace where the application will be deployed.
* `project`: The Argo CD project name.
* `destination`: The destination cluster and namespace for the application.
* `source`: The source Helm repository URL, chart, and version.
* `helm`:
  * `valuesObject`: A structured object that allows you to specify custom Helm values for the chart deployment. This configuration allows you to override default values and set specific parameters for the application.
* `syncPolicy`: Synchronization policies for Argo CD, including automated sync options.
* `awsSecretsManager`:
  * `enabled`: Enable or disable integration with AWS Secrets Manager.
  * `kind`: The type of secret store (`SecretStore` or `ClusterSecretStore`).
  * `refreshInterval`: Defines how often the secrets should be refreshed from AWS Secrets Manager.
  * `secretName`: The name of the Kubernetes secret that will be created or synchronized.
    * `data`: Specifies the mapping of Kubernetes secret keys to the external secret management keys in AWS Secrets Manager.
* `awsParameterStore`:
  * `enabled`: Enable or disable integration with AWS Parameter Store.
  * `kind`: The type of secret store (`SecretStore` or `ClusterSecretStore`).
  * `refreshInterval`: Defines how often the secrets should be refreshed from AWS Parameter Store.
  * `secretName`: The name of the Kubernetes secret that will be created or synchronized.
    * `data`: Specifies the mapping of Kubernetes secret keys to the external secret management keys in AWS Parameter Store.

#### Multiple Sources Support
There is the possibility to use multiple sources in the application definition. Here is an example of using multiple sources:

```yaml
sources:
  - repoURL: "https://charts.bitnami.com/bitnami"
    chart: "rabbitmq-cluster-operator"
    targetRevision: 3.13.0
    helm:
      valueFiles:
        - $values/projects/infra/rabbitmqClusterOperator/helm-values.yaml
  - repoURL: 'git@github.com:Configure8inc/c8-k8s-yahoo-infra.git'
    targetRevision: main
    ref: values
```

#### Including Additional Application Variables

The main application can include additional application variables or other configurations by specifying them in the sources. Here is an example:

```yaml
sources:
  - repoURL: 'git@github.com:Configure8inc/c8-k8s-yahoo-infra.git'
    targetRevision: main
    ref: values
  - repoURL: "git@github.com:Configure8inc/argo-app-of-apps.git"
    targetRevision: main
    path: "charts/app-of-apps"
    helm:
      valueFiles:
        - $values/projects/infra/awsEfsCsiDriver/app-values.yaml
        - $values/projects/infra/awsLoadBalancerController/app-values.yaml
        - $values/projects/infra/clusterAutoscaler/app-values.yaml
        - $values/projects/infra/externalSecrets/app-values.yaml
        - $values/projects/infra/lokiStack/app-values.yaml
        - $values/projects/infra/nginxIngressController/app-values.yaml
        - $values/projects/infra/rabbitmqClusterOperator/app-values.yaml
        - $values/projects/infra/redis/app-values.yaml
      ignoreMissingValueFiles: true
```

### External Secrets Support

Some applications support creating secrets using an external secret management system, such as AWS Secrets Manager, AWS Parameter Store, Azure Key Vault, or HashiCorp Vault, by leveraging the External Secrets Operator. This functionality allows you to synchronize secrets from external systems into your Kubernetes cluster, ensuring that sensitive data is managed securely and consistently.

Example Configuration for rabbitmqClusterOperator
Here is an example configuration in values.yaml that demonstrates how to deploy the `rabbitmqClusterOperator` with support for external secrets using AWS Secrets Manager and AWS Parameter Store:

```yaml
rabbitmqClusterOperator:
  enabled: true
  name: "rabbitmq-cluster-operator"
  namespace: myapp-use2-argocd
  project: myapp-use2-infrastructure
  destination:
    server: "https://kubernetes.default.svc"
    namespace: myapp-rabbitmq-use2
  source:
    path: ""
    repoURL: "https://charts.bitnami.com/bitnami"
    chart: "rabbitmq-cluster-operator"
    targetRevision: 3.13.0
  helm:
    valuesObject:
      clusterOperator:
        nodeSelector:
          C8App: "C8App"
  awsSecretsManager:
    enabled: true
    kind: ClusterSecretStore
    refreshInterval: 12h
    secretName: myapp-rabbitmq-secrets-use2
    data:
      - secretKey: username
        remoteRef:
          key: "RABBITMQ_USERNAME"
      - secretKey: password
        remoteRef:
          key: "RABBITMQ_PASSWORD"
  awsParameterStore:
    enabled: false  # AWS Parameter Store integration is disabled in this example
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

"You can find a complete example of usage in the `example` folder."