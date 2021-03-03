# Test Kubernetes Manifest

Assorted test Kubernetes applications that can be used for testing Tanzu Kubernetes Grid.

## Busybox

Deploy Pod
```
kubectl apply -f busybox.yaml
```

## Hello-World

Sample blue / green deployment with 2 versioned pods and services.  Ingress can be switched between v1.10 and v1.11.  Optional contour  httpproxy for weighted blue/green service.

Deploy  Pods and services
```
kubectl apply -f hello-blue.yaml
kubectl apply -f hello-green.yaml
```

Deploy v1.10 (Blue) Ingress
```
kubectl apply -f hello-ingress-blue.yaml
```

Swith Ingress to v1.11 (Green) Service
``` 
kubectl apply -f hello-ingress-green.yaml
```

Optional: If using Contour Ingress Control, deploy a weighted 80/20 Ingress for Blue/Green
```
kubectl apply -f hello-weighted-httpproxy.yaml
```

## Kuard

Simple

Pod
```
kubectl run --restart=Never --image=gcr.io/kuar-demo/kuard-amd64:blue kuard
kubectl port-forward kuard 8080:8080
```

Deployment
```
kubectl apply -f kuard/kuard.yaml
By default the mainifest is set to expose as nodeport to you can check the service using
kubectl get svc kuard
```


## Tinytools

Simple pod that includes ping, curl, wget, traceroute commands

Deploy using YAML
```
kubectl apply -f tinytools.yaml
```

Apply from Kubernetes CLI
```
kubectl run tinytools  --image=docker.io/giantswarm/tiny-tools -- sleep 360000
```

Exec into POD
```
kubectl exec -ti tinytools -- sh
```



