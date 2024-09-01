# Changelog

All notable changes to this project will be documented in this file.

## [1.0.1]

### Added
- **Support for External Secrets**:
  - Added support for creating secrets using external secret management systems, such as AWS Secrets Manager and AWS Parameter Store, leveraging the External Secrets Operator.
  - New configuration parameters `awsSecretsManager` and `awsParameterStore` added to `values.yaml` to enable and configure external secret synchronization.

- **New Applications**:
  - Added support for the following applications:
    - `external-secrets`: Synchronizes secrets from external secret management systems to Kubernetes.
    - `rabbitmq-cluster-operator`: Manages RabbitMQ clusters on Kubernetes with integrated support for external secrets.
    - `redis`: Deploys Redis with support for secure secret management.
  
### Changed
- **Unified Application-Specific Configuration**:
  - Updated `values.yaml` to include a unified configuration structure for all supported applications.
  - Enhanced configuration flexibility by supporting dynamic values for Helm deployments.
  - Included examples and improved documentation for deploying applications with Helm and Argo CD.

- **Improved Helm Template Logic**:
  - Updated Helm templates to include checks for `enabled` flags and dynamically manage resources based on `values.yaml`.
  - Added support for dynamically setting the `kind` field in the templates (`ClusterSecretStore` or `SecretStore`).

### Fixed
- **Namespace Creation**:
  - Added logic to Helm templates to create Kubernetes namespaces dynamically if not already present.

- **Finalizer Management**:
  - Ensured Argo CD-managed resources have the `resources-finalizer.argocd.argoproj.io` finalizer to prevent accidental deletions and ensure proper cleanup.

## [1.0.0] - Initial Release

### Added
- **Initial Set of Applications**:
  - `aws-efs-csi-driver`: AWS EFS CSI Driver for mounting EFS volumes in Kubernetes pods.
  - `aws-load-balancer-controller`: AWS Load Balancer Controller for managing AWS ELB/NLB resources.
  - `aws-node-termination-handler`: AWS Node Termination Handler for handling EC2 instance terminations gracefully.
  - `cluster-autoscaler`: Kubernetes Cluster Autoscaler for automatically adjusting the size of the cluster.
  - `metric-server`: Metrics Server for Kubernetes, provides resource metrics.
  - `nginx-ingress-controller`: Nginx Ingress Controller for managing external access to services in a Kubernetes cluster.
