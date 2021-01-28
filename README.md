# Getting Started Guide With ArgoCD

This guide helps you get started with ArgoCD. This is a simple guide
that takes you through the following steps:

* [Installing the ArgoCD Operator](#installing-the-argocd-operator)
* Installing an ArgoCD instance
* Deploying A Sample Application.


# Installing the ArgoCD Operator

The easiest way to install the ArgoCD Operator is via the OpenShift UI.

![install-argocd-operator](resources/images/install-argo-operator.gif)

To install it via the UI you simply...

* Click on `Operators` drop down on the leftside navigation.
* Click on `OperatorHub`
* In the search box type `argocd`.
* Select the `Argo CD` card (Note, that, this is a community supported operator).
* Click on `Continue` on the 'Show Community Operator' information notification.
* Click `Install` on the `Argo CD` installation dialog.

Another way to do this is to use the manifest directly. You can use the
resources in this repo to install the ArgoCD Operator:

```shell
oc apply -k https://github.com/RedHatWorkshops/argocd-getting-started/resources/manifests/argocd-operator-install
```

This uses [kustomize](https://kustomize.io/) to load the manifests needed
to install the Argo CD Operators. These 3 are:

* [argocd-namespace.yaml](resources/manifests/argocd-operator-install/argocd-namespace.yaml)
* [argocd-operatorgroup.yaml](resources/manifests/argocd-operator-install/argocd-operatorgroup.yaml)
* [argocd-subscription.yaml](resources/manifests/argocd-operator-install/argocd-subscription.yaml)
