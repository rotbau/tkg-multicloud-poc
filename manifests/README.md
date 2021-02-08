# Test Kubernetes Manifest

Assorted test Kubernetes applications that can be used for testing Tanzu Kubernetes Grid.

## Tinytools

Simple pod that includes ping, curl, traceroute commands

Deploy using YAML
```
kubectl apply -f tinytools.yaml
```

Apply from Kubernetes CLI
```
kubectl run tt  --image=docker.io/giantswarm/tiny-tools -- sleep 360000
```

Exec into POD
```
kubectl exec -ti tt -- sh
```
