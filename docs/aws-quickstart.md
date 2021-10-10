# AWS Quickstart

The following guide will install a GOKP cluster on your AWS Account.
Please read the entire guide before attempting and install.

# Prerequisites

The following are preqs. Since this is centered around [KIND](https://kind.sigs.k8s.io/) and [CAPI](https://cluster-api.sigs.k8s.io/), there will be a lot of similar prereqs:

These are things that are needed:

* AWS Account ([IAM Prereqs](https://cluster-api-aws.sigs.k8s.io/topics/iam-permissions.html#ec2-provisioned-kubernetes-clusters))
* GitHub Token
* Docker on your workstation

Podman may or maynot work. It's [considered experemental by KIND](https://kind.sigs.k8s.io/docs/user/rootless/#creating-a-kind-cluster-with-rootless-podman) so YMVM.
