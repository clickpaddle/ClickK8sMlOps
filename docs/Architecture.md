
# Kubernetes MLOps Platform: Complete Architecture

## Table of Contents

1. [Objective](#objective) ..................................................................... 1
2. [Global Architecture](#global-architecture) ........................................................... 1
3. [Key Components](#key-components) ................................................................ 2
4. [Deployment Steps](#deployment-steps) .............................................................. 2
   4.1 [Prepare Kubernetes](#prepare-kubernetes) ........................................................ 2
   4.2 [Install cert-manager](#install-cert-manager) ...................................................... 3
   4.3 [Upgrade Kustomize](#upgrade-kustomize) ......................................................... 3
   4.4 [Install Istio](#install-istio) ............................................................. 4
   4.5 [Install KServe](#install-kserve) ............................................................ 4
   4.6 [Verify Installation](#verify-installation) ....................................................... 5
   4.7 [Install Kubeflow](#install-kubeflow) .......................................................... 5
   4.8 [Deploy Sample Model (KServe)](#deploy-sample-model-kserve) .............................................. 6
   4.9 [Deployment Validation](#deployment-validation) ..................................................... 6
5. [Install Argo CD](#install-argo-cd) ............................................................... 7
6. [Monitoring and Observability](#monitoring-and-observability) .................................................. 7
7. [Multi-tenancy and Security](#multi-tenancy-and-security) .................................................... 8
8. [Standard MLOps Pipeline](#standard-mlops-pipeline) ....................................................... 8
9. [Final Checks](#final-checks) .................................................................. 9
10. [Best Practices](#best-practices) ................................................................ 9


1. Objective
Transform a basic Kubernetes cluster into a comprehensive MLOps platform integrating Kubeflow, KServe, ArgoCD/Argo Rollouts, monitoring, multi-tenancy, and security.
2. Global Architecture
Logical cluster structure:
Kubernetes Cluster
istio-system: istiod (Ingress / Routing)
cert-manager: TLS Certificates and Webhooks
kubeflow: Dashboard, Pipelines, Katib, Notebooks
kserve: Model management (InferenceServices)
ml: Production namespace for models
argocd: GitOps automation
monitoring: Prometheus, Grafana, Kiali
3. Key Components
Component
Role
Istio
Ingress, service mesh, routing for KServe & Kubeflow
Cert-manager
TLS management and webhook certificates
Kubeflow Pipelines
ML workflow orchestration (DAG)
Katib
AutoML and hyperparameter tuning
KServe
Highly scalable model deployment (Serving)
Argo CD
Continuous deployment via GitOps
Prometheus / Grafana
System and application metrics

4. Deployment Steps
4.1 Prepare Kubernetes
Ensure your cluster meets the following prerequisites:
Version $\ge$ v1.24
cluster-admin privileges
Dynamic storage configured (StorageClass)
4.2 Install cert-manager
# Installation de cert-manager
luc@trex:~$ kubectl apply -f [https://github.com/cert-manager/cert-manager/releases/latest/download/cert-manager.yaml](https://github.com/cert-manager/cert-manager/releases/latest/download/cert-manager.yaml)

# Vérification (doit être en statut Running)
luc@trex:~$ kubectl get pods -n cert-manager
pod/cert-manager-7844dc6b64-8q2st 1/1 Running 0 45s


4.3 Upgrade Kustomize
luc@trex:~$ mkdir -p /opt/automation/kustomize && cd /opt/automation/kustomize
luc@trex:~$ VERSION=v5.2.1
luc@trex:~$ wget [https://github.com/kubernetes-sigs/kustomize/releases/download/kustomize%2F$VERSION/kustomize](https://github.com/kubernetes-sigs/kustomize/releases/download/kustomize%2F$VERSION/kustomize)_{$VERSION}_linux_amd64.tar.gz
luc@trex:~$ tar -xvf kustomize_{$VERSION}_linux_amd64.tar.gz
luc@trex:~$ sudo mv kustomize /usr/local/bin/
luc@trex:~$ kustomize version


4.4 Install Istio
luc@trex:~$ curl -L [https://istio.io/downloadIstio](https://istio.io/downloadIstio) | sh -
luc@trex:~$ istioctl install --set profile=default -y


4.5 Install KServe
luc@trex:~$ kubectl apply -f [https://github.com/kserve/kserve/releases/latest/download/kserve.yaml](https://github.com/kserve/kserve/releases/latest/download/kserve.yaml)


4.6 Verify Installation
luc@trex:~$ kubectl get crd | grep inferenceservice
inferenceservices.serving.kserve.io 2026-01-20T12:00:00Z


4.7 Install Kubeflow
luc@trex:~$ git clone [https://github.com/kubeflow/manifests.git](https://github.com/kubeflow/manifests.git)
luc@trex:~$ cd manifests
luc@trex:~$ while ! kustomize build example | kubectl apply -f -; do sleep 10; done


4.8 Deploy Sample Model
Fichier iris.yaml :
apiVersion: serving.kserve.io/v1beta1
kind: InferenceService
metadata:
  name: iris
  namespace: ml
spec:
  predictor:
    sklearn:
      storageUri: gs://kfserving-examples/models/sklearn/1.0/model


4.9 Deployment Validation
luc@trex:~$ kubectl apply -f iris.yaml
luc@trex:~$ kubectl get inferenceservice iris -n ml


5. Install Argo CD
luc@trex:~$ kubectl create namespace argocd
luc@trex:~$ kubectl apply -n argocd -f [https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml](https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml)


6. Monitoring and Observability
Prometheus/Grafana : Métriques de performance.
Kiali : Visualisation du Service Mesh Istio.
7. Multi-tenancy and Security
Utilisation de Kubeflow Profiles pour l'isolation.
RBAC strict par namespace.
TLS automatique via cert-manager.
8. Standard MLOps Pipeline
Training : Kubeflow Pipelines.
Storage : Modèle poussé sur S3/GCS.
Deployment : Argo CD détecte le changement de manifest.
Serving : KServe expose l'InferenceService.
9. Final Checks
Vérifier que les Pods sont en état Running.
Valider l'Ingress Istio.
10. Best Practices
Isolation stricte Dev/Prod.
Utilisation du Scale-to-zero pour économiser les coûts.
Tests automatisés avant promotion.

