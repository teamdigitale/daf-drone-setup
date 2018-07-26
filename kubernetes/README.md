# Kubernetes

### Ingress

Contains the services exposed externally. To apply modifications run the following

```
$ kubectl apply -f kubernetes/drone-ingress.test.yml
```


### Persistent Volume [Claim]

Drone uses a volume claim of 10Gi on the VM volume.
To create the volume and the claim run the following:

```
$ kubectl apply -f kubernetes/drone-persistent-volume.yml
$ kubectl apply -f kubernetes/drone-persistent-volume-claim.yml
```
