# Docker Quickstart

The following guide will install a GOKP cluster on your local machine using docker.
Please read the entire guide before attempting and install.

# Prerequisites

The following are preqs. Since this is centered around [KIND](https://kind.sigs.k8s.io/) and [CAPI](https://cluster-api.sigs.k8s.io/), there will be a lot of similar prereqs. I recommend reading over them.

At a minimum:
* [GitHub Token](github-token.md)
* Docker on your workstation

Podman may or maynot work. It's [considered experemental by KIND](https://kind.sigs.k8s.io/docs/user/rootless/#creating-a-kind-cluster-with-rootless-podman) so YMVM.

Linux users should take note about [too many open files](https://kind.sigs.k8s.io/docs/user/known-issues/#pod-errors-due-to-too-many-open-files) issue, as you'll need to configure this.

# Installing the Cluster

After you have [gotten the binary](../README.md#getting-the-binary), you can run the `create-cluster` command. You will need your GitHub token.

```shell
gokp create-cluster development \
--cluster-name=$MYCLUSTER --github-token=$GH_TOKEN
```

Other options to note:

* `--ha` - This is `false` by default, a development cluster gives you 1 control-plane node and 2 workers. Set it to `true` to get a 3x3 cluster.
* `--gitops-controller` - By default this is set to `argocd`, but can also be set to `fluxcd`.
* `--private-repo` - By default this is set to `true`. Set it to `false` to create a public repo.

After about 40 min you should have a cluster ready to go. You'll have some information.

```shell
INFO[1037] Cluster Successfully installed! Everything you need is under: ~/.gokp/$MYCLUSTER
```

The Kubeconfig of this cluster is in this directory

```shell
export KUBECONFIG=~/.gokp/$MYCLUSTER/$MYCLUSTER.kubeconfig
```

Run `kubectl get pods -A` and you should see a Kuard smaple application.

```
$ k get pods -A | grep 'kuard'
NAMESPACE     NAME                                                  READY   STATUS    RESTARTS       AGE
kuard         kuard-857f95f9df-99x87                                1/1     Running   0              4m43s
```

Run `kubectl get nodes` and you should have 3 controlplane and 3 worker nodes.

```
NAME                            STATUS   ROLES                  AGE     VERSION
mycluster-control-plane-6tmwd     Ready    control-plane,master   6m57s   v1.22.1
mycluster-control-plane-jgfq9     Ready    control-plane,master   4m20s   v1.22.1
mycluster-control-plane-nsqdc     Ready    control-plane,master   6m21s   v1.22.1
mycluster-md-0-5657887c77-4f2pd   Ready    <none>                 6m36s   v1.22.1
mycluster-md-0-5657887c77-8g5ph   Ready    <none>                 6m36s   v1.22.1
mycluster-md-0-5657887c77-xj728   Ready    <none>                 6m36s   v1.22.1
```

# Adding Workloads

To add a workload, you can use the repo saved under `~/.gokp/$MYCLUSTER/$MYCLUSTER`.

```shell
$ cd ~/.gokp/$MYCLUSTER/$MYCLUSTER
```

> :exclamation: The SSH key used to make commits/pushes is under `~/.gokp/$MYCLUSTER` and it ends with an `_rsa`
> you will need to add this via `ssh-add` or set the [GIT_SSH_COMMAND](https://git-scm.com/docs/git) environment variable.

There is a sample of what you can do under `cluster/tenants`.

```shell
$ ls -1 cluster/tenants/
kuard
```

Create your workload in this directory with all the needed artifacts.
For example, to deploy NGINX sample workload. You would first create
the directory.

```shell
mkdir cluster/tenants/nginx
```

Then I would create the YAML files for NGINX in this new directory. First the namespace.

```shell
kubectl create ns nginx \
--dry-run=client -o yaml > cluster/tenants/nginx/nginx-ns.yaml
```

Next the deployment.

```shell
kubectl create deployment nginx -n nginx --image=nginx \
--dry-run=client -o yaml > cluster/tenants/nginx/nginx-deploy.yaml
```

Then the service.

```shell
kubectl create service clusterip nginx --tcp=80:8080 -n nginx \
-o yaml --dry-run=client > cluster/tenants/nginx/nginx-svc.yaml
```

> Optionally, you can also create your `Kustomization` files here.

Commit these to your repo.

```shell
git add .
git commit -am "added new application"
git push
```

In a few minutes you should see that your workload is on the cluster now.

```shell
kubectl get deploy,svc,pods -n nginx
NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx   1/1     1            1           60s

NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
service/nginx   ClusterIP   10.131.140.13   <none>        80/TCP    60s

NAME                         READY   STATUS    RESTARTS   AGE
pod/nginx-6799fc88d8-77tmk   1/1     Running   0          59s
```

# Cleaning Up

To delete your cluster just run the following.

```shell
gokp delete-cluster development \
--cluster-name=$MYCLUSTER --kubeconfig=/home/chernand/.gokp/$MYCLUSTER/$MYCLUSTER.kubeconfig
```

This will delete your cluster that's running locally on docker. It does not delete the repo that was created, so you will have to do that yourself.
