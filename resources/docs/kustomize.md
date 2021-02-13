# Working With Kustomize

In the [previous lab](../../README.md), you were able to deploy
some Kubernetes manifests and keep them in sync with your cluster
using ArgoCD. This is a powerful too, but can lead to a lot of YAML
being copied everywhere. In a lot of cases, you'd like to deploy an
application to multiple places with only minor changes. How do you do
that without copying YAML everywhere? This is where `kustomize` comes
in to save the day!

In this section, we'll be using [`kustomize`](https://kustomize.io/)
to show you how you can cut down on a lot of YAML duplication. You can
think of `kustomize` as a templeting engine. It's gotten so popular that
it's built into `kubectl` (and by extention `oc`)! See `kubectl apply
--help` for more info (you're looking for the `-k` option).

Before you get started, make sure you have the [ArgoCD Operator
installed](../../README.md#installing-the-argocd-operator) and an [ArgoCD
instance up](../../README.md#installing-an-argocd-instance) and running.
