# Flux CD Post Install Checks

Verify that Flux CD has been installed along with the Kuard sample app.

```shell
$ kubectl get pods -A | egrep 'flux-system|kuard'
NAMESPACE                           NAME                                                             READY   STATUS    RESTARTS      AGE
flux-system                         helm-controller-5d945d5b88-999hr                                 1/1     Running   0             8m29s
flux-system                         kustomize-controller-8647d466b6-q4xq5                            1/1     Running   0             8m26s
flux-system                         notification-controller-77f68bf8f4-mn6xw                         1/1     Running   0             8m23s
flux-system                         source-controller-7c8cd98868-9fzjs                               1/1     Running   0             8m21s
kuard                               kuard-857f95f9df-4scrm                                           1/1     Running   0             7m36s
```

There should be two `Kustomizations` on the cluster

```shell
$ kubectl get kustomizations.kustomize.toolkit.fluxcd.io -A
NAMESPACE     NAME          READY   STATUS                                                            AGE
flux-system   flux-system   True    Applied revision: main/a7813635ad03b16b7b005c35de58c98ef1719162   8m17s
flux-system   tenants       True    Applied revision: main/a7813635ad03b16b7b005c35de58c98ef1719162   7m48s
```

You should see your cluster's git repo configuration on the cluster as well.

```shell
kubectl get gitrepositories.source.toolkit.fluxcd.io -A
```

Using the `flux` cli, make sure you have connectivity. (this assumes you've exported your KUBECONFIG env var)

```shell
flux logs
```

Running a check is also recommended.

```shell
flux check
```

