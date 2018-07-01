# drone-poc

A Drone.io *proof-of-concept* for the Team Digitale


### Getting Kubernetes ready
An Ingress, a Persistent Volume and a Persistent Volume Claim must be created.
Instructions ![here](kubernetes/README.md)


### Installing and configuring Drone
Instructions ![here](drone-install/README.md)


### Exposing Drone
Forwarding port 8000 from the Drone server is required.
Instructions:

```
$ export POD_NAME=$(kubectl get pods -n default -l "component=server,app=drone,release=drone" -o jsonpath="{.items[0].metadata.name}")

$ kubectl -n default port-forward $POD_NAME 8000:8000
```

### Configuring the pipelines
Both the instructions and an example can be found in ![drone-pipeline](drone-pipeline/README.md)

Test `.drone.yml`: ![here](drone-pipeline/test/.drone.yml)

Build & deploy `.drone.yml`: ![here](drone-pipeline/build/.drone.yml)

