# drone-poc

A Drone.io *proof-of-concept* for the Team Digitale


## Kubernetes

### Ingress

Contains the services exposed externally. To apply modifications run the following

```
$ kubectl apply -f kubernetes/drone-ingress.test.yml
```

### Drone Volume Claim

Drone uses a volume claim of 10Gi on the vm volume.
To create the volume and the claim run the following:

```bash
$ kubectl apply -f kubernetes/drone-persistent-volume.yml
$ kubectl apply -f kubernetes/drone-persistent-volume-claim.yml
```


### Installing and configuring Drone



#### Step 4: exposing the pod
```
$ export POD_NAME=$(kubectl get pods -n default -l "component=server,app=drone,release=drone" -o jsonpath="{.items[0].metadata.name}")

$ kubectl -n default port-forward $POD_NAME 8000:8000
```
