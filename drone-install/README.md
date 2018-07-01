
# 1. install

```
$ helm install --name drone --namespace default -f drone-test.yml stable/drone

```





## OLD VERSION

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
