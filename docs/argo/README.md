# Argo CD Post Install Checks

Login to your Argo CD instance by performing the following:

Get the inital admin password

```shell
kubectl get secrets argocd-initial-admin-secret -n argocd  -o jsonpath='{.data.password}' | base64 -d ; echo
```

Access the Argo CD UI with the following

```shell
kubectl port-forward service/argocd-server 8080:443 -n argocd
```

You should be able to access the UI by visiting `http://127.0.0.1:8080`. There you'll see a few `Applications`. You should see one for each namespace. You should also see  one for `argocd` itself and another for `kuard` (a sample application).

This was deployed as `ApplicationSets`, take a look.

```shell
kubectl get appsets -n argocd
```

Which then creates `Applications`

```shell
kubectl get apps -n argocd
```

This is deployed via the Git repo it created, so check your account. Any changes you want to make should be done via the Git repo.

> NOTE: You might see some things "out of sync". The YAML Exporter is currently "best effort" so you may have to do some editing in your Git repo

