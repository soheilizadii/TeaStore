## Install OSM

OSM can be installed as follows:

```sh
export NAMESPACE=default # namespace which should be monitored

kubectl create ns osm-system
helm install osm osm --repo https://openservicemesh.github.io/osm --namespace osm-system
# If you use an older Kubernetes version, use OSM v0.9:
# helm install osm osm --repo https://openservicemesh.github.io/osm --namespace osm-system --version 0.9.2

sleep 60s

kubectl patch meshconfig osm-mesh-config -n osm-system -p '{"spec":{"traffic":{"enablePermissiveTrafficPolicyMode":true}}}'  --type=merge
kubectl patch meshconfig osm-mesh-config -n osm-system -p '{"spec":{"traffic":{"enableEgress":true}}}' --type=merge

kubectl label namespace $NAMESPACE openservicemesh.io/monitored-by=osm --overwrite
kubectl annotate namespace $NAMESPACE openservicemesh.io/metrics=enabled --overwrite
kubectl annotate namespace $NAMESPACE openservicemesh.io/sidecar-injection=enabled --overwrite
```

## Install Theodolite

As we are using the latest version of Theodolite, which has not been released yet, we need to clone Theodolite's repository and install it as follows:

```sh
helm install theodolite <path-to-theodolite> -f minimal.yaml
```

Or once everything has been released later:

```sh
helm install theodolite theodolite/theodolite -f minimal.yaml
```

## Setup Monitoring

```sh
kubectl apply -f pod-monitors.yaml
```

## Create ConfigMap for SUT Files

```sh
kubectl create configmap teastore-deployment --from-file=../kubernetes/teastore-clusterip-split/
```

## Create ConfigMap for Load Profile

```sh
kubectl create configmap teastore-jmeter-browse --from-file=../jmeter/teastore_browse_nogui.jmx
kubectl create configmap teastore-jmeter-deployment --from-file=jmeter.yaml
```

## Setup Benchmark

```sh
kubectl apply -f benchmark.yaml
```

## Start Benchmark Execution

```sh
kubectl apply -f execution-users.yaml
```

## Uninstall Everything

Uninstall OSM:

```sh
helm uninstall osm --namespace osm-system
kubectl delete ns osm-system
```

Uninstall Theodolite:

```sh
helm uninstall theodolite
```