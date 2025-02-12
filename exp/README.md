# Experimental(Technical Preview)

This is a technical preview of the experimental features that are currently being developed in the project. These features are not yet ready for production use and are subject to change.

# turtles-etcd-restore

## Setting up the environment

To set up the environment, navigate to the root of the repository and run:

```bash
export RANCHER_HOSTNAME="<hostname>"
export NGROK_API_KEY="<api-key>"
export NGROK_AUTHTOKEN="<api-authtoken>"
export USE_TILT_DEV=true (default)

make dev-env
```

**Note:** setting `USE_TILT_DEV` environment variable to `false` will result in manually deploying Rancher Turtles locally instead
of Tilt deployment and can be used for testing Rancher Turtles with Helm chart changes (enabling/disabling feature flags when passed as argument to Turtles helm installation command).

The `Makefile` target sets up the environment by executing the `scripts/turtles-dev.sh`
script with the `RANCHER_HOSTNAME` argument. Under the hood, it performs the following steps:

1. Creates a kind cluster.
2. Deploys cert-manager, CAPI Operator and Rancher Turtles.
3. Deploys CAPRKE2 provider.
4. Deploys Docker provider.
5. Deploys ngrok.
6. Deploys Rancher accessible via ngrok.

Environment is prepared for cluster creation using CAPRKE2. UI is accessible via `RANCHER_HOSTNAME`.

## Creating a cluster

To deploy an RKE2 cluster with automatic snapshots enabled:

```bash
export CONTROL_PLANE_MACHINE_COUNT=1
export WORKER_MACHINE_COUNT=1
export CLUSTER_NAME=rke2
export KUBERNETES_VERSION=v1.30.4
export RKE2_VERSION=v1.30.4+rke2r1

# Prevent replacing lb-config variables by envsubst
export address='$address'
export server='$server'

envsubst < test/e2e/data/cluster-templates/docker-rke2.yaml | kubectl apply -f -
```

## Performing the snapshot and restore

When all machines in the cluster are ready, automatic ETCDMachineSnapshot object should appear on the management cluster soon.

```bash

kubectl get etcdmachinesnapshot -A
```

To perform a restore run the following command:

```bash
export CLUSTER_NAMESPACE=default
export CLUSTER_NAME=rke2
export ETCD_MACHINE_SNAPSHOT_NAME="<snapshot_name_from_the_output>"

envsubst < exp/etcdrestore/examples/etcd-restore.yaml | kubectl apply -f -
```

## Cleanup

To clean up the environment, run the following command from the root of the repo:

```bash
make clean-dev-env
```