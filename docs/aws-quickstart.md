# AWS Quickstart

The following guide will install a GOKP cluster on your AWS Account.
Please read the entire guide before attempting and install.

# Prerequisites

The following are preqs. Since this is centered around [KIND](https://kind.sigs.k8s.io/) and [CAPI](https://cluster-api.sigs.k8s.io/), there will be a lot of similar prereqs. I recommend reading over them.

At a minimum:
* AWS Account ([IAM Prereqs](https://cluster-api-aws.sigs.k8s.io/topics/iam-permissions.html#ec2-provisioned-kubernetes-clusters))
* [GitHub Token](github-token.md)
* Docker on your workstation

Podman may or maynot work. It's [considered experemental by KIND](https://kind.sigs.k8s.io/docs/user/rootless/#creating-a-kind-cluster-with-rootless-podman) so YMVM.

Linux users should take note about [too many open files](https://kind.sigs.k8s.io/docs/user/known-issues/#pod-errors-due-to-too-many-open-files) issue, as you'll need to configure this.

# Installing the Cluster

After you have [gotten the binary](../README.md#getting-the-binary), you can run the `create-cluster` command. You will need your AWS Access Key, Secret Key, the SSH Keypair to use ([already must exist](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/ec2/import-key-pair.html)), and your GitHub token.

```shell
gokp create-cluster --cluster-name=$MYCLUSER \
--github-token=$GH_TOKEN \
--aws-ssh-key=$AWS_SSH_KEY_NAME \
--aws-access-key=$AWS_ACCESS_KEY_ID \
--aws-secret-key=$AWS_SECRET_ACCESS_KEY \
--private-repo=true
```

> NOTE: A private repo will commit your GitHub Token to the repo. If you don't want this set `--private-repo` to `false`

After about 40 min you should have a cluster ready to go. You'll have some information.

```shell
INFO[1037] Cluster Successfully installed! Everything you need is under: ~/.gokp/$MYCLUSTER
```

The Kubeconfig of this cluster is in this directory

```shell
export KUBECONFIG=~/.gokp/$MYCLUSTER/$MYCLUSTER.kubeconfig
```

Run `kubectl get pods -A` and you should see ArgoCD pods along with a Kuard smaple application.

```
$ k get pods -A | egrep 'argocd|kuard'
NAMESPACE     NAME                                                  READY   STATUS    RESTARTS       AGE
argocd        argocd-application-controller-0                       1/1     Running   0              4m37s
argocd        argocd-applicationset-controller-5856984b4c-wj8s8     1/1     Running   0              4m39s
argocd        argocd-dex-server-64f95f8c77-2wzcj                    1/1     Running   0              4m39s
argocd        argocd-redis-978c79c75-l9c6p                          1/1     Running   0              4m39s
argocd        argocd-repo-server-85d6c7c8c4-jqwdz                   1/1     Running   0              4m39s
argocd        argocd-server-7f86787cf-s5sh9                         1/1     Running   0              4m39s
kuard         kuard-857f95f9df-99x87                                1/1     Running   0              4m43s
```

Run `kubectl get nodes` and you should have 3 controlplane and 3 worker nodes.

```
NAME                          STATUS   ROLES                  AGE   VERSION
ip-10-0-155-27.ec2.internal   Ready    control-plane,master   11m   v1.22.2
ip-10-0-254-40.ec2.internal   Ready    control-plane,master   12m   v1.22.2
ip-10-0-69-120.ec2.internal   Ready    <none>                 12m   v1.22.2
ip-10-0-71-144.ec2.internal   Ready    control-plane,master   10m   v1.22.2
ip-10-0-77-205.ec2.internal   Ready    <none>                 12m   v1.22.2
ip-10-0-85-76.ec2.internal    Ready    <none>                 12m   v1.22.2
```

Login to your Argo CD instance

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

# Adding Workloads

To add a workload, you can use the repo saved under `~/.gokp/$MYCLUSTER/$MYCLUSTER`. 

> Or you can just `git clone` it to another directory if you wish.

```
$ cd ~/.gokp/$MYCLUSTER/$MYCLUSTER
```

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

In a few minutes you should see your application running on the cluster.

```shell
$ kubectl get application nginx -n argocd
NAME    SYNC STATUS   HEALTH STATUS
nginx   Synced        Healthy
```

Workload is on the cluster now.

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
gokp delete-cluster \
--cluster-name=$MYCLUSTER \
--kubeconfig=/home/chernand/.gokp/$MYCLUSTER/$MYCLUSTER.kubeconfig
```

This will delete your cluster running in AWS. It does not delete the repo that was created, so you will have to do that yourself.
