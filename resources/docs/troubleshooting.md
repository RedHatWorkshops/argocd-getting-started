# Troubleshooting

I've written down some of the most common issues you may run into when
doing this getting started guide.

## No Matches for Kind ArgoCD

If you've tried to apply the ArgoCD instance manifest in the [Installing
an ArgoCD Instance](../../README.md#installing-an-argocd-instance) section, and got the following error:

```
error: unable to recognize "https://github.com/RedHatWorkshops/argocd-getting-started/resources/manifests/argocd-instance": no matches for kind "ArgoCD" in version "argoproj.io/v1alpha1
```

You can try the following:

* Make sure you've [installed the ArgoCD Operator](../../README.md#installing-the-argocd-operator)
* Wait for the CRD to become available.

The easiest way to wait is to run an `until` loop:

```shell
until oc apply -k https://github.com/RedHatWorkshops/argocd-getting-started/resources/manifests/argocd-instance
do
   sleep 3
done
```
