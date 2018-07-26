# Install Drone

**DO NOT USE THE drone-test.yml FILE AT THE MOMENT!**

## 1. Configure a Kubernetes secret with the GitHub secret 
```
$ kubectl create secret generic drone-server-secrets  \
    --namespace=default \
    --from-literal=DRONE_GITHUB_SECRET="OUR_GITHUB_SECRET"
```


## 2. Install Drone
```
helm install --name drone --namespace default \
  --set rbac.create=false \
  --set ingress.enabled=false \
  --set persistence.existingClaim="drone-pv-claim" \
  --set server.env.DRONE_PROVIDER="github" \
  --set server.env.DRONE_ESCALATE="plugins/docker" \
  --set server.env.DRONE_GITHUB="true" \
  --set server.env.DRONE_OPEN="true" \
  --set server.env.DRONE_ORGS="DroneTestTeamDigitale" \
  --set server.env.DRONE_GITHUB_CLIENT="OUR_GITHUB_CLIENT" \
  --set server.envSecrets.drone-server-secrets[0]=DRONE_GITHUB_SECRET \
  stable/drone
```


## 3. Configure the DinD entrypoint
Unfortunately, some problems with cloning the repositories (related to the MTU), appear if we don't change the default Docker-in-Docker entrypoint in the Drone agent.

We need to change the current deployment.
```helm get drone > agent-deployment.yaml```

Delete everything from `agent-deployment.yaml` except for the `# Source: drone/templates/deployment-agent.yaml` section.
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

