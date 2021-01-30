---
layout: post
title:  "MongoDB on K8s"
sub_title: "Baking MongoDB on K8s"
date:   2020-11-02 12:00:00 -0600
categories: mongodb kubernetes k8s 
---

## Create mongodb namespace

All our MongoDB K8s objects will be deployed into this namespace.

```bash
kubectl create namespace mongodb
```

## Apply MongoDB Custom Resource Definitions

[Custom Resource Definitions](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/) are a way to extend the K8s API and the MongoDB Operator uses them to define the following MongoDB objects.

* [MongoDB](https://docs.mongodb.com/kubernetes-operator/stable/reference/k8s-operator-specification/) - K8s resource for MongoDB objects such as Standalone, ReplicaSet and ShardedClusters
* MongoDBUser - K8s resource for MongoDB users
* [MongoDBOpsManager](https://docs.mongodb.com/kubernetes-operator/stable/reference/k8s-operator-om-specification/) - K8s resource for MongoDB Enterprise Ops Manager

```bash
# Download Custom Resource Definitions
curl -O https://raw.githubusercontent.com/mongodb/mongodb-enterprise-kubernetes/master/crds.yaml
kubectl apply -f crds.yaml
```

## Deploy MongoDB Operator

Once CRDs are loaded apply the MongoDB Operator to create service accounts and the actual Operator in our K8s cluster.

```bash
# Download MongoDB Enterprise Operator
curl -O https://raw.githubusercontent.com/mongodb/mongodb-enterprise-kubernetes/master/mongodb-enterprise.yaml
kubectl apply -f mongodb-enterprise.yaml
# View the Operator deployment
kubectl -n mongodb describe deployment mongodb-enterprise-operator
kubectl -n mongodb get deployment mongodb-enterprise-operator










## References

1. [MongoDB Enterprise Kubernetes Operator](https://docs.mongodb.com/kubernetes-operator/stable/)