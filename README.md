# drone-poc


## kubernetes

### Ingress

Contains the services exposed externally. To apply modifications run the following

```
$ kubectl apply -f kubernetes/drone-ingress.test.yml
```

### Drone Volume Claim

Drone uses a volume claim of 10Gi on the vm volume.
To create the volume and the claim run the following:

```
$ kuebctl apply -f kubernetes/drone-persistent-volume.yml
$ kubectl apply -f kubernetes/drone-persistent-volume-claim.yml
```
