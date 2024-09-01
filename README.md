# Helm Chart for Argo CD Applications

This Helm chart allows you to deploy multiple applications using Argo CD. It supports various Kubernetes resources, including AWS Load Balancer Controller, Nginx Ingress Controller, Cluster Autoscaler, and others, providing a flexible and customizable deployment process.

| Supported Application           | Description                                                                                     |
|---------------------------------|-------------------------------------------------------------------------------------------------|
| `aws-efs-csi-driver`            | AWS EFS CSI Driver for mounting EFS volumes in Kubernetes pods.                                 |
| `aws-load-balancer-controller`  | AWS Load Balancer Controller for managing AWS ELB/NLB resources.                                 |
| `aws-node-termination-handler`  | AWS Node Termination Handler for handling EC2 instance terminations gracefully.                 |
| `cluster-autoscaler`            | Kubernetes Cluster Autoscaler for automatically adjusting the size of the cluster.              |
| `external-secrets`              | External Secrets Operator for syncing secrets from external secret management systems to Kubernetes. |
| `metric-server`                 | Metrics Server for Kubernetes, provides resource metrics.                                       |
| `nginx-ingress-controller`      | Nginx Ingress Controller for managing external access to services in a Kubernetes cluster.       |
| `rabbitmq-cluster-operator`     | RabbitMQ Cluster Operator for managing RabbitMQ clusters on Kubernetes.                         |
| `redis`                         | Redis deployment for caching and data storage in Kubernetes environments.                       |

## Overview

This Helm chart is designed to deploy multiple applications managed by Argo CD in a Kubernetes cluster. It supports various controllers and drivers, such as AWS Load Balancer Controller, Nginx Ingress Controller, Cluster Autoscaler, AWS EFS CSI Driver, and AWS Node Termination Handler.

## Installation

To install the Helm chart, add this Helm repository and use the `helm install` command:

```bash
helm repo add my-helm-repo https://your-helm-repo-url
helm repo update
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
Each application has its own set of configurable parameters in the values.yaml file. Below are the common parameters:

* enabled: Enable or disable the application deployment.
* name: The name of the application.
* namespace: The Kubernetes namespace where the application will be deployed.
* project: The Argo CD project name.
* destination: The destination cluster and namespace for the application.
* source: The source Helm repository URL, chart, and version.
* helm:
  * valuesObject: A structured object that allows you to specify custom Helm values for the chart deployment. This configuration allows you to override default values and set specific parameters for the application.
* syncPolicy: Synchronization policies for Argo CD, including automated sync options.
* awsSecretsManager:
  * enabled: Enable or disable integration with AWS Secrets Manager.
  * kind: The type of secret store (SecretStore or ClusterSecretStore).
  * refreshInterval: Defines how often the secrets should be refreshed from AWS Secrets Manager.
  * secretName: The name of the Kubernetes secret that will be created or synchronized.
    * data: Specifies the mapping of Kubernetes secret keys to the external secret management keys in AWS Secrets Manager.
* awsParameterStore:
  * enabled: Enable or disable integration with AWS Parameter Store.
  * kind: The type of secret store (SecretStore or ClusterSecretStore).
  * refreshInterval: Defines how often the secrets should be refreshed from AWS Parameter Store.
  * secretName: The name of the Kubernetes secret that will be created or synchronized.
    * data: Specifies the mapping of Kubernetes secret keys to the external secret management keys in AWS Parameter Store.

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

### Full example chart deployment using Argo CD:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-use2-apps-with-apps
  namespace: myapp-use2-argocd
  finalizers:
    - resources-finalizer.argocd.argoproj.io
spec:
  project: myapp-use2-infrastructure
  destination:
    server: "https://kubernetes.default.svc"
    namespace: kube-system
  source:
    repoURL: "git@github.com:Configure8inc/argo-app-of-apps.git" # Replace with your Git repo URL
    targetRevision: main  # Specify the branch, tag, or commit hash
    path: "charts/app-of-apps"
    helm:
      valuesObject:
        namePrefix: "myapp-"
        nameSuffix: "-use2"

        metricsServer:
          enabled: true
          name: metrics-server
          namespace: myapp-use2-argocd
          project: myapp-use2-infrastructure
          source:
            repoURL: "https://kubernetes-sigs.github.io/metrics-server/"
            targetRevision: "3.11.0"
            chart: metrics-server
            helm:
              valuesObject:
                replicas: 1
          syncPolicy:
            automated:
              prune: true
              selfHeal: true

        awsLoadBalancerController:
          enabled: true
          name: "aws-load-balancer-controller"
          namespace: myapp-use2-argocd
          project: myapp-use2-infrastructure
          source:
            repoURL: "https://aws.github.io/eks-charts"
            targetRevision: 1.6.2
            chart: aws-load-balancer-controller
            helm:
              valuesObject:
                clusterName: c8-eks-cluster-myapp-use2
                createIngressClassResource: true
                region: us-east-2
                vpcId: vpc-0ac15f3b191287be0
                defaultTargetType: ip
                enableWaf: true
                enableWafv2: true

                nodeSelector:
                  C8App: "C8App"

                replicaCount: 2
                serviceAccount:
                  create: true
                  name: myapp-use2-aws-load-balancer-controller
                  annotations:
                    eks.amazonaws.com/role-arn: arn:aws:iam::123456789101:role/myapp-prod-aws-load-balancer-controller

                resources:
                  limits:
                    cpu: 300m
                    memory: 128Mi
                  requests:
                    cpu: 100m
                    memory: 128Mi
                updateStrategy:
                  type: RollingUpdate
                  rollingUpdate:
                    maxSurge: 1
                    maxUnavailable: 1
          ingresses:
            - name: myapp-external-use2
              namespace: myapp-ingress-nginx-use2
              annotations:
                alb.ingress.kubernetes.io/scheme: internet-facing
                alb.ingress.kubernetes.io/target-type: ip
                alb.ingress.kubernetes.io/group.name: myapp-external-use2
                alb.ingress.kubernetes.io/load-balancer-name: myapp-external-use2
                alb.ingress.kubernetes.io/tags: business_unit=myapp,team=DevOps,owner=configure8,ManagedBy=Argo,application=c8
                alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:us-east-2:123456789101:certificate/19009c7e-eab5-4e18-8aa8-5c367a92276c
                alb.ingress.kubernetes.io/listen-ports: '[{"HTTP": 80}, {"HTTPS":443}]'
                alb.ingress.kubernetes.io/actions.ssl-redirect: '{"Type": "redirect", "RedirectConfig": { "Protocol": "HTTPS", "Port": "443", "StatusCode": "HTTP_301"}}'
                alb.ingress.kubernetes.io/healthcheck-path: /healthz
                alb.ingress.kubernetes.io/load-balancer-attributes: deletion_protection.enabled=false
                alb.ingress.kubernetes.io/ssl-policy: ELBSecurityPolicy-TLS13-1-2-2021-06
              rules:
                - http:
                    paths:
                      - pathType: ImplementationSpecific
                        path: /*
                        backend:
                          service:
                            name: ssl-redirect
                            port:
                              name: use-annotation
                - host: 'myapp.configure8.io'
                  http:
                    paths:
                      - pathType: ImplementationSpecific
                        path: /*
                        backend:
                          service:
                            name: myapp-nginx-ingress-use2-ingress-nginx-controller
                            port:
                              name: http
                - http:
                    paths:
                      - pathType: ImplementationSpecific
                        path: /*
                        backend:
                          service:
                            name: myapp-nginx-ingress-use2-ingress-nginx-controller
                            port:
                              name: http

            - name: myapp-internal-use2
              namespace: myapp-ingress-nginx-use2
              annotations:
                alb.ingress.kubernetes.io/scheme: internal
                alb.ingress.kubernetes.io/target-type: ip
                alb.ingress.kubernetes.io/group.name: myapp-internal-use2
                alb.ingress.kubernetes.io/load-balancer-name: myapp-internal-use2
                alb.ingress.kubernetes.io/tags: business_unit=myapp,team=DevOps,owner=configure8,ManagedBy=Argo,application=c8
                alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:us-east-2:123456789101:certificate/19009c7e-eab5-4e18-8aa8-5c367a92276c
                alb.ingress.kubernetes.io/inbound-cidrs: 10.100.0.0/20
                alb.ingress.kubernetes.io/listen-ports: '[{"HTTP": 80}, {"HTTPS":443}]'
                alb.ingress.kubernetes.io/actions.ssl-redirect: '{"Type": "redirect", "RedirectConfig": { "Protocol": "HTTPS", "Port": "443", "StatusCode": "HTTP_301"}}'
                alb.ingress.kubernetes.io/healthcheck-path: /healthz
                alb.ingress.kubernetes.io/load-balancer-attributes: deletion_protection.enabled=false
                alb.ingress.kubernetes.io/ssl-policy: ELBSecurityPolicy-TLS13-1-2-2021-06

              rules:
                - http:
                    paths:
                      - pathType: ImplementationSpecific
                        path: /*
                        backend:
                          service:
                            name: ssl-redirect
                            port:
                              name: use-annotation
                - http:
                    paths:
                      - pathType: ImplementationSpecific
                        path: /*
                        backend:
                          service:
                            name: myapp-use2-nginx-ingress-nginx-controller
                            port:
                              name: http


          syncPolicy:
            automated:
              prune: true
              selfHeal: true

        nginxIngressController:
          enabled: true
          name: "nginx-ingress"
          namespace: myapp-use2-argocd
          project: myapp-use2-infrastructure
          destination:
            server: "https://kubernetes.default.svc"
            namespace: myapp-ingress-nginx-use2
          source:
            repoURL: "https://kubernetes.github.io/ingress-nginx"
            targetRevision: 4.11.2
            chart: ingress-nginx
            helm:
              valuesObject:
                controller:
                  resources:
                    limits:
                      cpu: 200m
                      memory: 384Mi
                    requests:
                      cpu: 50m
                      memory: 384Mi
                  ingressClassResource:
                    name: myapp-use2-ingress-nginx
                    enabled: true
                  ingressClassByName: true
                  publishService:
                    enabled: true
                  service:
                    type: NodePort
                    nodePorts:
                      http: 32080
                      https: 32443
                    annotations:
                      service.beta.kubernetes.io/aws-load-balancer-backend-protocol: "http"
                      service.beta.kubernetes.io/aws-load-balancer-connection-idle-timeout: "3600"
                  config:
                    use-forwarded-headers: "true"
                    use-gzip: true

                  kind: Deployment
                  replicaCount: 2

                  nodeSelector:
                    C8App: "C8App"

                  updateStrategy:
                    type: RollingUpdate
                    rollingUpdate:
                      maxUnavailable: 0

                  podSecurityContext:
                    runAsUser: 65534
                    runAsGroup: 65534
                    seccompProfile:
                      type: RuntimeDefault
          syncPolicy:
            automated:
              prune: true
              selfHeal: true
            syncOptions:
              - CreateNamespace=true

        clusterAutoscaler:
          enabled: true
          name: "cluster-autoscaler"
          namespace: myapp-use2-argocd
          project: myapp-use2-infrastructure
          source:
            repoURL: "https://kubernetes.github.io/autoscaler"
            targetRevision: 9.37.0
            chart: cluster-autoscaler
            helm:
              valuesObject:
                autoDiscovery:
                  clusterName: "c8-eks-cluster-myapp-use2"
                replicaCount: 2
                awsRegion: us-east-2
                resources:
                  limits:
                    cpu: 100m
                    memory: 300Mi
                  requests:
                    cpu: 30m
                    memory: 300Mi
                rbac:
                  create: true
                  serviceAccount:
                    name: myapp-use2-cluster-autoscaler
                    annotations: 
                      eks.amazonaws.com/role-arn: "arn:aws:iam::123456789101:role/c8-myapp-prod-cluster-autoscaler-role"
                nodeSelector:
                  C8App: "C8App"
          syncPolicy:
            automated:
              prune: true
              selfHeal: true
            

        awsNodeTerminationHandler:
          enabled: true
          name: "aws-node-termination-handler"
          namespace: myapp-use2-argocd
          project: myapp-use2-infrastructure
          destination:
            server: "https://kubernetes.default.svc"
            namespace: kube-system
          source:
            repoURL: "https://aws.github.io/eks-charts"
            chart: aws-node-termination-handler
            targetRevision: 0.21.0
          syncPolicy:
            automated:
              prune: true
              selfHeal: true

        awsEfsCsiDriver:
          enabled: true
          name: "aws-efs-csi-driver"
          namespace: myapp-use2-argocd
          project: myapp-use2-infrastructure
          source:
            repoURL: "https://kubernetes-sigs.github.io/aws-efs-csi-driver"
            targetRevision: 3.0.8
            chart: aws-efs-csi-driver
            helm:
              valuesObject:
                controller:
                  serviceAccount:
                    create: true
                    name: myapp-use2-efs
                    annotations:
                      eks.amazonaws.com/role-arn: arn:aws:iam::123456789101:role/c8-myapp-prod-efs-role
                  resources:
                    limits:
                      cpu: 200m
                      memory: 256Mi
                    requests:
                      cpu: 50m
                      memory: 256Mi

                replicaCount: 1
                nodeSelector:
                  C8App: "C8App"

                storageClasses:
                  - name: efs-sc
                    provisioner: efs.csi.aws.com
                    mountOptions:
                      - tls
                    parameters:
                      provisioningMode: efs-ap
                      fileSystemId: fs-0fbecb095672982a4
                      directoryPerms: "700"
                      gidRangeStart: "1000"
                      gidRangeEnd: "2000"
                      basePath: "/dynamic_provisioning"
          syncPolicy:
            automated:
              prune: true
              selfHeal: true
      
        externalSecrets:
          enabled: true
          name: "external-secrets"
          namespace: myapp-use2-argocd 
          project: myapp-use2-infrastructure
          destination:
            server: "https://kubernetes.default.svc"
            namespace: myapp-external-secrets-use2
          source:
            path:
            repoURL: "https://charts.external-secrets.io"
            targetRevision: 0.9.11
            chart: external-secrets
            helm:
              valuesObject:
                serviceAccount:
                  create: true
                  automount: true
                  annotations:
                    eks.amazonaws.com/role-arn: "arn:aws:iam::123456789101:role/c8-myapp-prod-external-secrets-role"
                  name: "myapp-external-secrets-use2"

                podSecurityContext:
                  runAsNonRoot: true
                  runAsUser: 1000
                  runAsGroup: 1000

                webhook:
                  podSecurityContext:
                    runAsNonRoot: true
                    runAsUser: 1000
                    runAsGroup: 1000

                certController:
                  podSecurityContext:
                    runAsNonRoot: true
                    runAsUser: 1000
                    runAsGroup: 1000

                nodeSelector:
                  C8App: "C8App"
          
          stores:
            - kind: ClusterSecretStore
              name: aws-parameter-store
              provider:
                aws:
                  service: ParameterStore
                  region: us-east-2
                  auth:
                    jwt:
                      serviceAccountRef:
                        name: myapp-external-secrets-use2
                        namespace: myapp-external-secrets-use2

          syncPolicy:
            automated:
              prune: true
              selfHeal: true
            syncOptions:
              - CreateNamespace=true
        
        redis:
          enabled: true
          name: "redis"
          namespace: myapp-use2-argocd
          project: myapp-use2-infrastructure
          destination:
            server: "https://kubernetes.default.svc"
            namespace: myapp-redis-use2
          source:
            path:
            repoURL: registry-1.docker.io/bitnamicharts
            chart: redis
            targetRevision: 19.3.2
            helm:
              valuesObject:
                global:
                  defaultStorageClass: "efs-sc"
                image:
                  tag: 7.2.4-debian-12-r16
                architecture: replication
                master:
                  count: 1
                  resources:
                    requests:
                      cpu: 0.5
                      memory: 512Mi
                    limits:
                      cpu: 1
                      memory: 512Mi
                  podSecurityContext:
                    enabled: true
                    fsGroupChangePolicy: Always
                    sysctls: []
                    supplementalGroups: []
                    fsGroup: 1001
                  containerSecurityContext:
                    enabled: true
                    seLinuxOptions: {}
                    runAsUser: 1001
                    runAsGroup: 1001
                    runAsNonRoot: true
                    allowPrivilegeEscalation: false
                    readOnlyRootFilesystem: true
                    seccompProfile:
                      type: RuntimeDefault
                    capabilities:
                      drop: ["ALL"]
                  nodeSelector:
                    C8App: "C8App"
                  persistence:
                    enabled: true
                    storageClass: efs-sc

                replica:
                  kind: StatefulSet
                  replicaCount: 2
                  resources:
                    requests:
                      cpu: 0.5
                      memory: 512Mi
                    limits:
                      cpu: 1
                      memory: 512Mi
                  podSecurityContext:
                    enabled: true
                    fsGroupChangePolicy: Always
                    sysctls: []
                    supplementalGroups: []
                    fsGroup: 1001
                  containerSecurityContext:
                    enabled: true
                    seLinuxOptions: {}
                    runAsUser: 1001
                    runAsGroup: 1001
                    runAsNonRoot: true
                    allowPrivilegeEscalation: false
                    readOnlyRootFilesystem: true
                    seccompProfile:
                      type: RuntimeDefault
                    capabilities:
                      drop: ["ALL"]
                  nodeSelector:
                    C8App: "C8App"
                  persistence:
                    enabled: true
                    storageClass: efs-sc
                  autoscaling:
                    enabled: false
                auth:
                  existingSecret: myapp-redis-secrets-use2

          awsParameterStore:
            enabled: true
            kind: ClusterSecretStore
            refreshInterval: 12h
            secretName: myapp-redis-secrets-use2
            data:
              - secretKey: redis-password
                remoteRef:
                  key: "REDIS_PASSWORD"
          syncPolicy:
            automated:
              prune: true
              selfHeal: true
            syncOptions:
              - CreateNamespace=true

        rabbitmqClusterOperator:
          enabled: true
          name: "rabbitmq-cluster-operator"
          namespace: myapp-use2-argocd
          project: myapp-use2-infrastructure
          destination:
            server: "https://kubernetes.default.svc"
            namespace: myapp-rabbitmq-use2
          source:
            path:
            repoURL: "https://charts.bitnami.com/bitnami"
            chart: "rabbitmq-cluster-operator"
            targetRevision: 3.13.0
          helm:
            valuesObject:
              clusterOperator:
                nodeSelector:
                  C8App: "C8App"
          awsParameterStore:
            enabled: true
            kind: ClusterSecretStore
            refreshInterval: 12h
            secretName: myapp-redis-secrets-use2
            data:
              - secretKey: username
                remoteRef:
                  key: "RABBITMQ_USERNAME"
              - secretKey: password
                remoteRef:
                  key: "RABBITMQ_PASSWORD"
          syncPolicy:
            automated:
              prune: true
              selfHeal: true
            syncOptions:
              - CreateNamespace=true

  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```
