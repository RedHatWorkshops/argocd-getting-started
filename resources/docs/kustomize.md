# Working With Kustomize

In the [previous lab](../../README.md), you were able to deploy
some Kubernetes manifests and keep them in sync with your cluster
using ArgoCD. This is a powerful too, but can lead to a lot of YAML
being copied everywhere. In a lot of cases, you'd like to deploy an
application to multiple places with only minor changes. How
do you do that without copying YAML everywhere? This is where
[`kustomize`](https://kustomize.io/) comes
in to save the day!

Before you get started, make sure you have the [ArgoCD Operator
installed](../../README.md#installing-the-argocd-operator) and an [ArgoCD
instance up](../../README.md#installing-an-argocd-instance) and running.

In this section we'll be going over:

* [Creating An App With Kustomize](#creating-an-app-with-kustomize)
* [Deploying An App With Kustomize](#deploying-an-app-with-kustomize)

# Creating An App With Kustomize

In the previous lab, we [deployed a sample application](../../README.md#deploying-a-sample-application), which deployed the app with a blue square. But what if you wanted a green square without copying all the configuration files? Use Kustomize!

# Deploying An App With Kustomize
