# Pre-reqs

 * [Install Google Cloud SDK](https://cloud.google.com/sdk/)

 * Authenticate to your GCP

   `gcloud init`

 * Install kubectl

   `gcloud components install kubectl`
    
 * [Install Helm](https://docs.helm.sh/using_helm/#installing-helm)
  
 * Check your setup

   `gcloud container --project $PROJECT clusters get-credentials $CLUSTER_NAME --zone $ZONE`
   `kubectl config current-context`

# Using the Helm Chart on GKE

 * Deploy a GKE cluster

```
gcloud container clusters create scylladb \
  --cluster-version 1.10.5-gke.4 \
  --num-nodes 4 \
  --machine-type n1-standard-8 \
  --image-type UBUNTU \
  --disk-type pd-ssd \
  --disk-size 50
```

Available options can be checked with this command:

`gcloud container get-server-config`

 * Configure basic RBAC

   `kubectl create -f scylladb/rbac-config.yaml`

 * Deploy scylla using Helm

   `helm init --upgrade --service-account tiller`

 * Check that the tiller is created

   `kubectl get serviceaccount -n kube-system | grep tiller`

helm install scylladb --namespace $whateverYouWant

 * Export an environment variable for the release

   `export RELEASE=$(helm list | grep scylladb | tr -s \t | cut -f1)`

 * Check the status 

   `helm status $RELEASE`


# Playing with Scylla

  * Check your scylla cluster

    `kubectl exec -ti $RELEASE-scylladb-0 -- nodetool status # Checking cluster status through the first node`

    `kubectl logs $RELEASE-scylladb-0 # Check the logs for 1st pod`

  * Grow your cluster by upgrading your Release - adding 2 more nodes. This will update the REVISION number on your release

    `helm upgrade --set replicaCount=5, $RELEASE helm-chart-scylla/`

    `helm history $RELEASE`

    `kubectl exec -ti $RELEASE-scylladb-0 -- nodetool status`

  * Shrink your cluster by upgrading your Release - removing one node

    `helm upgrade --set replicaCount=4, $RELEASE helm-chart-scylla/`

    `helm history $RELEASE`

    `kubectl exec -ti $RELEASE-scylladb-0 -- nodetool status`

  * Playing with loads:

    `kubectl exec -ti $RELEASE-scylladb-3 -- /bin/bash`

    `nodetool decommission`

    `cassandra-stress write duration=30s -rate threads=36 -node <ip>`


  * Or you can play with our tutorial [here](https://www.scylladb.com/2017/11/30/mutant-monitoring-system-day-1/) and [here](https://www.scylladb.com/2018/01/18/mms-day-2-building-tracking-system/).

  * Shrink your cluster by rolling back to REVISION 1 - removing another node

    `helm rollback $RELEASE 1`

    `helm history $RELEASE`

    `kubectl exec -ti $RELEASE-scylladb-0 -- nodetool status`

  * Delete your helm release

    `helm delete $RELEASE --purge`
