# Getting Started Guide With ArgoCD

This guide helps you get started with ArgoCD. This is a simple guide
that takes you through the following steps:

* [Installing the ArgoCD Operator](#installing-the-argocd-operator)
* [Installing an ArgoCD Instance](#installing-an-argocd-instance)
* [Deploying A Sample Application](#deploying-a-sample-application)


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

> :bulb: **NOTE**: You don't have to use `kustomize`. You can `oc apply -f` these files individually

Once you've installed the Operator, you can now install an ArgoCD Instance.

# Installing an ArgoCD Instance

Once the Operator is installed, we need to tell it to deploy an instance
of ArgoCD. First you need to make sure ArgoCD has the ability to
administer the cluster.

First, you create a `ClusterRoleBinding` that gives the `ServiceAcocunt`
named `argocd-application-controller`, the `cluster-admin`
`ClusterRole`. This allows ArgoCD to manage the cluster.

```yaml
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: argocd-application-controller-cluster-admin
subjects:
  - kind: ServiceAccount
    name: argocd-application-controller
    namespace: argocd
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
```


Now you can install ArgoCD using the following manifest:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ArgoCD
metadata:
  name: argocd
  namespace: argocd
spec:
  server:
    route:
      enabled: true
  dex:
    openShiftOAuth: true
    image: quay.io/redhat-cop/dex
    version: v2.22.0-openshift
  resourceCustomizations: |
    route.openshift.io/Route:
      ignoreDifferences: |
        jsonPointers:
        - /spec/host
  rbac:
    defaultPolicy: ''
    policy: |
      g, system:cluster-admins, role:admin
    scopes: '[groups]'
  initialRepositories: |
    - name:  argocd-getting-started
      type: git
      url: https://github.com/RedHatWorkshops/argocd-getting-started
```

This manifest does some customization to work with OpenShift:

* Creates a route (under: `.spec.server.route.enabled`)
* Uses Dex to allow the mapping of OpenShift groups to ArgoCD groups (under: `.spec.dex`)
* RBAC mapping of the `system:cluster-admins` OCP group to `role:admin` role in ArgoCD (under: `.spec.rbac`)
* Initialize this repo for use later (under: `.spec.initialRepositories`)

For more info, please see [the official ArgoCD Operator
Doc](https://argocd-operator.readthedocs.io/en/latest/reference/argocd/)

You can install both of these [using this
repo](resources/manifests/argocd-instance) by running:

```shell
oc apply -k https://github.com/RedHatWorkshops/argocd-getting-started/resources/manifests/argocd-instance
```

> :bulb: You can **optionally** apply the [argocd-instance.yaml](resources/manifests/argocd-instance/argocd-instance.yaml) and [argocd-cluster-role.yaml ](resources/manifests/argocd-instance/argocd-cluster-role.yaml ) one at a time.

Once applied, you should see the following in the `argocd` namespace,
when you run the `oc get pods -n argocd` command:

```
$ oc get pods -n argocd
NAME                                             READY   STATUS    RESTARTS   AGE
argocd-application-controller-774cc495f6-wd58h   1/1     Running   0          80m
argocd-dex-server-5c549895f9-6srhw               1/1     Running   0          80m
argocd-operator-67dbb7db5f-p7cbg                 1/1     Running   0          81m
argocd-redis-6f7cfddbcb-c5kd5                    1/1     Running   0          80m
argocd-repo-server-5454d6c459-sltk2              1/1     Running   0          80m
argocd-server-5d998f7668-qn62f                   1/1     Running   0          80m
```

Once installed, get the route for the ArgoCD UI:

```shell
oc get route argocd-server -n argocd -o jsonpath='{.spec.host}{"\n"}'
```

You should be presented with this.

![argocd-login](resources/images/argocd-login.png)

Go ahead and click `LOGIN VIA OPENSHIFT`, and login as `kubeadmin`
(or a user that has the `cluster-admin` role).

You should see this screen:

![argocd](resources/images/argocd.png)

# Deploying A Sample Application

In [this repo](resources/manifests/bgdk-yaml), we have some manifesets
that you can use to test. This is a simple app that includes:

* [A Namespace](resources/manifests/bgdk-yaml/bgd-namespace.yaml)
* [A Deployment](resources/manifests/bgdk-yaml/bgd-deployment.yaml)
* [A Service](resources/manifests/bgdk-yaml/bgd-svc.yaml)
* [A Route](resources/manifests/bgdk-yaml/bgd-route.yaml)

Collectively, this is known as an `Application` within ArgoCD. Therefore,
you must define it as such in order to apply these manifest in your
cluster.

Here is the `Application` manifest we are going to use:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: bgdk-app
  namespace: argocd
spec:
  destination:
    namespace: argocd
    server: https://kubernetes.default.svc
  project: default
  source:
    path: resources/manifests/bgdk-yaml
    repoURL: https://github.com/RedHatWorkshops/argocd-getting-started
    targetRevision: main
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
  sync:
    comparedTo:
      destination:
        namespace: argocd
        server: https://kubernetes.default.svc
      source:
        path: resources/manifests/bgdk-yaml
        repoURL: https://github.com/RedHatWorkshops/argocd-getting-started
        targetRevision: main
```

Let's break this down a bit.

* ArgoCD's concept of a `Project` is different than OpenShift's. Here you're installing the application in ArgoCD's `default` project (`.spec.project`). **NOT** OpenShift's `default` project.
* The destination server is the server we installed ArgoCD on (noted as `.spec.destination.server`).
* The manifest repo where the YAML resides and the path to look for the YAML is under `.spec.source`.
* The `.spec.syncPolicy` is set to automatically sync the repo.
* The last section `.spec.sync` just says what are you comparing the repo to. (Basically "Compare the running config to the desired config")

The `Application` CR (`CustomResource`) can be applied using [this repo](resources/manifests/bgdk-app) by running:

```shell
oc apply -k https://github.com/RedHatWorkshops/argocd-getting-started/resources/manifests/bgdk-app
```

This should create the `bgdk-app` in the ArgoCD UI.

![bgdk-app](resources/images/bgdk-app.png)
