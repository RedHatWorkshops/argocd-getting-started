# Getting Started Guide

This guide helps you get started with the OpenShift GitOps Operaotr. This
is a simple guide that takes you through the following steps:

* [Installing the Operator](#installing-the-openshift-gitops-operator)
* [Setting up ArgoCD](#setting-up-argocd)
* [Deploying A Sample Application](#deploying-a-sample-application)

The idea of this guide is that it should work on ANY OpenShift 4.7+
cluster. So you'll need access to an OpenShift cluster or CRC. You will
also need the `oc` cli utility.

# Installing the OpenShift GitOps Operator

The easiest way to install the OpenShift GitOps Operator is via the
OpenShift UI.

You can install the Operator via the UI in the Administrator Perspective:

* Click on `Operators` drop down on the leftside navigation.
* Click on `OperatorHub`
* In the search box type `openshift gitops`.
* Select the `Red Hat OpenShift GitOps` card.
* Click `Install` on the `Red Hat OpenShift GitOps` installation dialog.
* Accept all the defaults in the `Install Operator` page and click `Install`

Another way to do this is to use the manifest directly. You can use the
resources in this repo to install the OpenShift GitOps Operator:

```shell
oc apply -k https://github.com/RedHatWorkshops/argocd-getting-started/resources/manifests/operator-install
```

This uses [kustomize](https://kustomize.io/) to load the manifests needed to install the OpenShift GitOps Operator. There is only 1 that is needed:

* [openshift-gitops-operator-sub.yaml](resources/manifests/operator-install/openshift-gitops-operator-sub.yaml)

> :bulb: **NOTE**: You don't have to use `kustomize`. You can `oc apply -f` the file individually.

The Operator is a "meta" Operator that installs both the ArgoCD Operator
and Instance; and the Tekton Operator and Instance.

Verify the installation by running `oc get pods -n openshift-gitops`. You should see the following output

```shell
$ oc get pods -n openshift-gitops
NAME                                                    READY   STATUS    RESTARTS   AGE
argocd-cluster-application-controller-6f548f74b-48bvf   1/1     Running   0          54s
argocd-cluster-redis-6cf68d494d-9qqq4                   1/1     Running   0          54s
argocd-cluster-repo-server-85b9d68f9b-4hj52             1/1     Running   0          54s
argocd-cluster-server-78467b647-8lcv9                   1/1     Running   0          54s
cluster-86f8d97979-lfdhv                                1/1     Running   0          56s
kam-7ff6f58c-2jxkm                                      1/1     Running   0          55s
```

> :heavy_exclamation_mark: **NOTE**: It'll take some time so you may want to run `watch oc get pods -n openshift-gitops`

# Setting up ArgoCD

Once the Operator is installed, we need to make some customizations for
this lab. First, we need to patch the manifest so that ArgoCD will ignore
router differences (since every route will be differnet).

```shell
oc patch argocd argocd-cluster -n openshift-gitops --type=json \
-p='[{"op": "add", "path": "/spec/resourceCustomizations", "value":"route.openshift.io/Route:\n  ignoreDifferences: |\n    jsonPointers:\n    - /spec/host\n"}]'
```

Next, you need to give the ArgoCD Service account permission to
make changes to your cluster. In practice, you will scope this to a
specific namespace or a set of access. For this lab we will give it
`cluster-admin`.

```shell
oc adm policy add-cluster-role-to-user cluster-admin -z argocd-cluster-argocd-application-controller -n openshift-gitops
```

Delete the ArgoCD server pod so that it can relaunch with the new set of permissions.

```shell
oc delete pods -l app.kubernetes.io/name=argocd-cluster-server -n openshift-gitops
```

The Operator installs the password in a secret. Extract this password to use to login to the ArgoCD instance.

```shell
oc extract secret/argocd-cluster-cluster -n openshift-gitops --to=-
```

To get the route for the ArgoCD UI:

```shell
oc get route argocd-cluster-server -n openshift-gitops -o jsonpath='{.spec.host}{"\n"}'
```

Once you visit the URL in your browser, you should be presented with
something that looks like this.

![argocd-login](resources/images/argocd-login.png)

Go ahead and login as `admin` with the password you've extracted above.

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
  namespace: openshift-gitops
spec:
  destination:
    namespace: openshift-gitops
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
        namespace: openshift-gitops
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

Clicking on this takes you to the overview page. You may see it as still progressing or full synced. 

![synced-app](resources/images/synced-app.png)

> :heavy_exclamation_mark: **NOTE**: You may have to click on `show hidden resources` on this page to see it all

At this point the application should be up and running. You can see
all the resources created with the `oc get pods,svc,route -n bgd`
command. The output should look like this:

```
$ oc get pods,svc,route -n bgd
NAME                       READY   STATUS    RESTARTS   AGE
pod/bgd-788cb756f7-kz448   1/1     Running   0          10m

NAME          TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service/bgd   ClusterIP   172.30.111.118   <none>        8080/TCP   10m

NAME                           HOST/PORT                                PATH   SERVICES   PORT   TERMINATION   WILDCARD
route.route.openshift.io/bgd   bgd-bgd.apps.example.com          bgd        8080                 None
```

Your output will be slightly different.

Visit your application by running the following to get the URL:

```shell
oc get route bgd -n bgd -o jsonpath='{.spec.host}{"\n"}'
```

Your application should look like this.

![bgd](resources/images/bgd.png)

Let's introduce a change! Patch the live manifest to change the color
of the box from blue to green:

```shell
oc -n bgd patch deploy/bgd --type='json' \
-p='[{"op": "replace", "path": "/spec/template/spec/containers/0/env/0/value", "value":"green"}]'
```

If you quickly (I'm not kidding, you have to be lightning fast or you'll
miss it) look at the screen you'll see it out of sync:

![outofsync](resources/images/out-of-sync.png)

But ArgoCD sees that difference and changes it back to the desired
state. Preventing drift.

![fullysynced](resources/images/fullysynced.png)

> :bulb: **NOTE**: If you're having touble catching the sync. Run your browser window and terminal window side-by-side. This is a good thing, that Argo acts so quickly. :smiley:

# Conclusion

As you can see, you can use ArgoCD to deploy you application and hinder
platform drift. This isn't only applicable to applications, but to other
cluster resources as well (e.g MachineSets, Authentication, AutoScaling,
etc).

This was a smiple demo that should get you familiar with using ArgoCD. If
you'd like to dip your toes in a little more advanced topics, feel free
to take a look at one of these modules:

* [Working with Kustomize](resources/docs/kustomize.md)
* [Syncwaves and Hooks](resources/docs/syncwaves-and-hooks.md) * **Coming Soon** *

There are more demos and videos that you can find on
[demo.openshift.com](https://demo.openshift.com/en/dev/argocd/)
