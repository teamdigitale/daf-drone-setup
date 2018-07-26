# Installing, configuring and exposing Drone

**DO NOT USE THE drone-test.yml FILE AT THE MOMENT!**

## 1. Configure a Kubernetes secret with the GitHub secret 
You will need to create this secret in GitHub as per the ![Drone docs](http://readme.drone.io/admin/setup-github/).
```
$ kubectl create secret generic drone-server-secrets  \
    --namespace=default \
    --from-literal=DRONE_GITHUB_SECRET="YOUR_GITHUB_SECRET"
```


## 2. Install Drone
Once you have created the secret, you will need to install Drone.
We will:
* disable RBAC, since our test environment doesn't use it
* disable Ingress, since we have created our own and don't rely on third-party ones such as GKE's
* link Drone to the Persistent Volume Claim we created
* set Drone to use GitHub, to let the Docker plugin run in privileged mode (needed), to be open to the members of our organization
* hardcode the GitHub client code and link to the secret

```
helm install --name drone --namespace default \
  --set rbac.create=false \
  --set ingress.enabled=false \
  --set persistence.existingClaim="drone-pv-claim" \
  --set server.env.DRONE_PROVIDER="github" \
  --set server.env.DRONE_ESCALATE="plugins/docker" \
  --set server.env.DRONE_GITHUB="true" \
  --set server.env.DRONE_OPEN="true" \
  --set server.env.DRONE_ORGS="YOUR_GITHUB_ORG" \
  --set server.env.DRONE_GITHUB_CLIENT="YOUR_GITHUB_CLIENT" \
  --set server.envSecrets.drone-server-secrets[0]=DRONE_GITHUB_SECRET \
  stable/drone
```


## 3. Exposing Drone
Forwarding port 8000 from the Drone server is required in order to access Drone's web UI.
Two commands have to be run from the command line:
```
$ export POD_NAME=$(kubectl get pods -n default -l "component=server,app=drone,release=drone" -o jsonpath="{.items[0].metadata.name}")

$ kubectl -n default port-forward $POD_NAME 8000:8000 &
```

Now Drone can be accessed at localhost:8000 (which is also forwarded to the external internet through the Ingress we created).


## 4. Configure the DinD entrypoint
Unfortunately, some problems with cloning the repositories appear if we don't change the default Docker-in-Docker entrypoint in the Drone agent. These problems seem to be related to Docker's MTU.

We need to change the current deployment. Let's get it:
```helm get drone > drone-deployment.yaml```
This file contains all of the configurations and the "inner" deployments of our `drone` deployment. We need to change the agent deployment.
Delete everything from `drone-deployment.yaml` except for the `# Source: drone/templates/deployment-agent.yaml` section.
Here, in `spec > template > spec > containers`, edit `drone-drone-dind` so that it matches this:

```yaml
- name: drone-drone-dind
    image: "docker.io/library/docker:17.12.0-ce-dind"
    imagePullPolicy: IfNotPresent
    env:
    - name: DOCKER_DRIVER
        value: overlay2
    command: ["/bin/sh"]
    args: ["-c", "iptables -N DOCKER-USER; iptables -I DOCKER-USER -p tcp --tcp-flags SYN,RST SYN -j TCPMSS --clamp-mss-to-pmtu; dockerd --host=unix:///var/run/docker.sock --host=tcp://127.0.0.1:2375 --mtu=1350"]
```

Then run `kubectl apply -f agent-deployment.yaml`.
This will result in a correct cloning step
![](https://i.imgur.com/ot55jn8.png)

otherwise, cloning the repository would fail
![](https://i.imgur.com/dqFdj8E.png)


Now Drone is working and ready to be used with the pipelines we created!