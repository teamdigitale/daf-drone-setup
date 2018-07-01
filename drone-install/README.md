# Install Drone
## 1. Configure a Kubernetes secret with the GitHub secret 
```
$ kubectl create secret generic drone-server-secrets  \
    --namespace=default \
    --from-literal=DRONE_GITHUB_SECRET="OUR_GITHUB_SECRET"
```


## 2. Install Drone using Helm
```
$ helm install --name drone --namespace default -f drone-test.yml stable/drone
```

Done!
