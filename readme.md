# ArgoCD Documentation

## Introduction

ArgoCD is a declarative, GitOps continuous delivery tool for Kubernetes. It automates the deployment of the desired application states in the specified target environments.

## Features

- **Declarative GitOps**: Manage your applications using Git as the source of truth.
- **Automated Sync**: Automatically sync your applications to the desired state.
- **Multi-Cluster Support**: Manage applications across multiple Kubernetes clusters.
- **Health Assessment**: Monitor the health of your applications and environments.
- **Rollback**: Easily rollback to previous application versions.

## Installation

To install ArgoCD, follow these steps:

1. **Install ArgoCD CLI**:

   ```sh
   brew install argocd
   ```

2. **Install ArgoCD in Kubernetes**:
   ```sh
   kubectl create namespace argocd
   kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
   ```

## Getting Started

1. **Access the ArgoCD API Server**:

   ```sh
   kubectl port-forward svc/argocd-server -n argocd 8080:443
   ```

2. **Login to ArgoCD**:

   ```sh
   argocd login localhost:8080
   ```

3. **Create a New Application**:

   ```sh
   argocd app create <app-name> \
   --repo <repo-url> \
   --path <app-path> \
   --dest-server https://kubernetes.default.svc \
   --dest-namespace <namespace>
   ```

4. **Sync the Application**:
   ```sh
   argocd app sync <app-name>
   ```

## Documentation and Support

For more detailed documentation, visit the [official ArgoCD documentation](https://argo-cd.readthedocs.io/). For support, join the [ArgoCD community](https://argoproj.github.io/community/).

## Conclusion

ArgoCD simplifies the process of managing and deploying applications in Kubernetes using GitOps principles. It provides powerful features to ensure your applications are always in the desired state.

## Using ArgoCD with Cert-Manager and Cloudflare

### Overview

In this section, we will cover how to integrate ArgoCD with Cert-Manager and Cloudflare for managing TLS certificates.

### Prerequisites

- A Kubernetes cluster
- ArgoCD installed
- Cert-Manager installed
- Cloudflare account and API token

### Steps

1. **Install Cert-Manager**:

   ```sh
   kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.5.3/cert-manager.yaml
   ```

2. **Configure Cloudflare DNS Challenge**:
   Create a secret with your Cloudflare API token:

   ```sh
   kubectl create secret generic cloudflare-api-token-secret --from-literal=api-token=<your-cloudflare-api-token> -n cert-manager
   ```

3. **Create a ClusterIssuer**:

   ```yaml
   apiVersion: cert-manager.io/v1
   kind: ClusterIssuer
   metadata:
     name: cloudflare-clusterissuer
   spec:
     acme:
       server: https://acme-v02.api.letsencrypt.org/directory
       email: <your-email>
       privateKeySecretRef:
         name: cloudflare-acme-private-key
       solvers:
         - dns01:
             cloudflare:
               email: <your-cloudflare-email>
               apiTokenSecretRef:
                 name: cloudflare-api-token-secret
                 key: api-token
   ```

4. **Create a Certificate Resource**:

   ```yaml
   apiVersion: cert-manager.io/v1
   kind: Certificate
   metadata:
     name: argo-cert
     namespace: argocd
   spec:
     secretName: argo-tls-cert
     issuerRef:
       name: cloudflare-clusterissuer
       kind: ClusterIssuer
     dnsNames:
       - argocd.<your-domain>
   ```

5. **Update ArgoCD IngressRoute**:
   Ensure your ArgoCD IngressRoute is configured to use the certificate:
   ```yaml
   apiVersion: traefik.io/v1alpha1
   kind: IngressRoute
   metadata:
     name: argocd-server
     namespace: argocd
   spec:
     entryPoints:
       - websecure
     routes:
       - kind: Rule
         match: Host(`argocd.<your-domain>`)
         services:
           - name: argocd-server
             port: 80
     tls:
       secretName: argo-tls-cert
   ```

### Debugging Tips

1. **Check ArgoCD Logs**:

   ```sh
   kubectl logs -n argocd -l app.kubernetes.io/name=argocd-server
   ```

2. **Check Cert-Manager Logs**:

   ```sh
   kubectl logs -n cert-manager -l app=cert-manager
   ```

3. **Verify Certificate Status**:

   ```sh
   kubectl describe certificate argo-cert -n argocd
   ```

4. **Check IngressRoute Status**:
   ```sh
   kubectl describe ingressroute argocd-server -n argocd
   ```

### Conclusion

Integrating ArgoCD with Cert-Manager and Cloudflare allows you to automate the management of TLS certificates, ensuring secure communication for your applications. Use the provided debugging tips to troubleshoot any issues that may arise during the setup.
