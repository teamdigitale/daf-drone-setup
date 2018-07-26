# drone-poc

A Drone.io *proof-of-concept* for the Team Digitale

## Abstract
The goal of this project is to implement Continuous Integration and Continuous Delivery in the ![DAF](https://teamdigitale.governo.it/it/projects/daf.htm).

DAF's complex architecture relies on microservices in the form of Docker containers, run on a Kubernetes cluster managed via Rancher.

As of now, the deployment process is far from being simple: tests must be run manually each time new code is committed, Docker images must be built by hand and the Kubernetes deployment requires running the kubectl apply command.

What we want to do is automating this process using CI/CD, in particular Drone.io.


## Steps to be taken

### Getting Kubernetes ready
An Ingress must be created in order to expose Drone; a Persistent Volume and a Persistent Volume Claim must be created to achieve persistence.
Instructions ![here](kubernetes/README.md)


### Installing and configuring Drone
Drone has to be installed, configured and exposed on our Kubernetes cluster.
Instructions ![here](drone-install/README.md)


### Configuring the pipelines
Both the instructions and an example can be found in ![drone-pipeline](drone-pipeline/README.md)

Test `.drone.yml`: ![here](drone-pipeline/test/.drone.yml). This pipeline runs tests and sends Slack notifications about them.

Build & deploy `.drone.yml`: ![here](drone-pipeline/build/.drone.yml). This pipeline builds the service's Docker image and updates the Kubernetes deployment to match the newly created version.

