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

```bash
$ kubectl apply -f kubernetes/drone-persistent-volume.yml
$ kubectl apply -f kubernetes/drone-persistent-volume-claim.yml
```


### Installing and configuring Drone

#### Step 1: installing Drone
```
$ helm install --name drone \
    --namespace default \
    --set ingress.enabled=false \
    --set rbac.create=false \
    --set persistence.existingClaim="drone-pv-claim" \
    stable/drone
```

#### Step 2: configuring the Github Secret
```
$ kubectl create secret generic drone-server-secrets  \
    --namespace=default \
    --from-literal=DRONE_GITHUB_SECRET="OUR_GITHUB_SECRET"
```

#### Step 3: configuring Drone
```
$ helm upgrade drone \
    --reuse-values \
    --set persistence.existingClaim="drone-pv-claim" \
    --set server.env.DRONE_PROVIDER="github" \
    --set server.env.DRONE_GITHUB="true" \
    --set server.env.DRONE_OPEN=true \
    --set server.env.DRONE_ORGS="DroneTestTeamDigitale" \
    --set server.env.DRONE_GITHUB_CLIENT="OUR_GITHUB_CLIENT" \
    --set server.envSecrets.drone-server-secrets[0]=DRONE_GITHUB_SECRET \
    stable/drone
```

#### Step 4: exposing the pod
```
$ export POD_NAME=$(kubectl get pods -n default -l "component=server,app=drone,release=drone" -o jsonpath="{.items[0].metadata.name}")
$ kubectl -n default port-forward $POD_NAME 8000:8000
```