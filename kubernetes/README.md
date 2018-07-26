# Kubernetes

### Ingress

A Kubernetes Ingress is required in order to expose Drone @ drone.teamdigitale.it.
The Ingress can be created by running the following:
```
$ kubectl apply -f kubernetes/drone-ingress.test.yml
```


### Persistent Volume [Claim]
Drone uses a Volume Claim of 10Gi on the VM storage in order to have persistence.
To create the Volume and the Claim run the following:
```
$ kubectl apply -f kubernetes/drone-persistent-volume.yml
$ kubectl apply -f kubernetes/drone-persistent-volume-claim.yml
```
