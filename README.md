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



## Pipeline

### Test

We will configure some automatic tests using [this](https://github.com/teamdigitale/daf-srv-storage) repository. Tests are run by executing `sbt test`. 

We simply need to create a *test* pipeline in order to run this test and check whether they run successfully.

First of all, let's activate the repository in Drone:

![img](https://i.imgur.com/VBvY5UB.png)

And let's add this `.drone.yml` file to the repository

```yaml
pipeline:
  build:
    image: hseeberger/scala-sbt
    commands:
      - sbt test
```

As we can see, the configuration is extremely simple and relies on a [Docker image](https://hub.docker.com/r/hseeberger/scala-sbt/) containing Scala and SBT.

Once the file is in the repository, every commit will trigger the test pipeline, and the results will be clear in Drone's web-UI

![img](https://i.imgur.com/bIWlCNY.png) 



Of course, we can also read the logs to check every step that was taken

![img](https://i.imgur.com/Opp5jd7.png)



### Build and Deploy

* https://github.com/teamdigitale/daf-models/tree/master/outbox-classification/web-api
* http://ml-api.daf.teamdigitale.test/
* https://github.com/teamdigitale/daf-models/blob/master/outbox-classification/web-api/kube-deploy-test.yml