# DAF Drone Setup

A Drone.io *proof-of-concept* for the Team Digitale.

## Abstract

The goal of this document is defining how to implement Continuous Integration (CI) and Continuous Delivery (CD) [1] for the [Data and Analytics Framework](https://teamdigitale.governo.it/it/projects/daf.htm) (DAF). In CI developers merge their changes back to the main branch as often as possible, while the goal of CD is to make sure that we can release new changes to customers quickly and in a sustainable way. Both approaches should help developers in their work and firmly avoid putting constraints or pains on them. CI/CD should be easy to implement and to update basing on feedbacks received from customers.

The [Data & Analytics Framework (DAF, in short)](https://pianotriennale-ict.readthedocs.io/en/latest/doc/09_data-analytics-framework.html) is an open source project developed in the context of the activities planned by the Italian [Three-Year Plan for ICT in Public Administration 2017 - 2019](https://pianotriennale-ict.readthedocs.io/en/latest/), approved by the Italian Government in 2017.

The DAF project is an attempt to establish a central Chief Data Officer (CDO) for the Government and Public Administration. Its main goal is to promote data exchange among Italian Public Administrations (PAs), to support the diffusion of open data, and to enable data-driven policies. The framework is composed by three building blocks:

- A Big Data Platform, to store in a unique repository the data of the PAs, implementing ingestion procedures to promote standardization and therefore interoperability among them. It exposes functionalities common to the Hadoop ecosystem, a set of (micro) services designed to improve data governance and a number of end-user tools that have been integrated with them.
- A Team of Data Experts (Data Scientists and Data Engineers), able to manage and evolve the platform and to provide support to PA on their analytics and data management activities in a consultancy fashion.
- A Regulatory Framework, that institutionalizes this activity at government level, and gives the proper mandate to the PA that will manage the DAF, in compliance with privacy policy.

Many of the components of the DAF are deployed as microservices that run on a [Kubernetes cluster](https://kubernetes.io/) backed by [Rancher](https://rancher.com/).
As of now, the deployment process is far from being simple: tests must be run manually each time new code is committed, Docker images must be built by hand and the Kubernetes deployment requires running the `kubectl apply -f file.yml` command.

What we want to do is defining an automated process for CI/CD. A detailed analysis of challenges and solutions applied can be found [here](https://docs.google.com/document/d/1Xi3MglejhG_tBD4qmx8wAqZ77c7OptrqHgRq4H7K878/edit?usp=sharing).


## The components involved
There are lots of components involved in the process; here we will try and clarify what they do, hoping this helps seeing the big picture of this project.
* **Drone**: [Drone](https://github.com/drone/drone) is a Docker-native CI/CD tool that we will use to automate testing, building and deploying of DAF's microservices
* **Kubernetes**: Kubernetes is an open-source container-orchestration system for automating deployment, scaling and management of containerized applications. In our case, Kubernetes is used to run many of DAF's components
* **Rancher**: Rancher is a container management platform that makes it easy to use and configure Kubernetes clusters, letting the user check logs, create Deployments, and much more through a simple UI
* **Sonatype Nexus**: Nexus is a repository manager used to organize, store and distribute software components. In our case, Nexus hosts the Docker images you would usually upload on the Docker Hub.

Two simple diagrams are here used to show how the components talk to each other.
![](https://i.imgur.com/uWWaSlk.jpg)
![](https://i.imgur.com/zDfw0WI.png)



## Installation and Setup

### Kubernetes Requirements

In general to configure drone in kubernetes we need:
- an Ingress entry;
- a Persistent Volume and a Persistent Volume Claim for data and configurations persistence.

Detailed instructions can be found [here](kubernetes/README.md).


### Drone Configurations
A tutorial for this step can be found [here](drone-install/README.md).


### Configuring the pipelines
Both the instructions and examples can be found in [drone-pipeline](drone-pipeline/README.md)

- [Test `.drone.yml`](drone-pipeline/test/.drone.yml). This pipeline runs tests and sends Slack notifications about them.

- [Build & deploy `.drone.yml`](drone-pipeline/build/.drone.yml). This pipeline builds the service's Docker image and updates the Kubernetes deployment to match the newly created version.


## How to change the organization and other settings
Let's assume that, at the moment, you are using persistence (which is regulated by Drone's Helm chart) and your configuration is open to every person in the "foobar" organization. What if you wanted to disable persistence and make Drone available only to one person?

Of course, uninstalling and reinstalling Drone doesn't look like a smart move. Instead, you can just upgrade the application to change some settings. For example, to disable persistence and allow access to you only, you would need to create a new client/secret pair from your GitHub account's Apps section, and use:

```
# delete the old secret
$ kubectl delete secret drone-server-secrets

# create the new secret 
$ kubectl create secret generic drone-server-secrets \
  --namespace=default \
  --from-literal=DRONE_GITHUB_SECRET="YOUR_NEW_GITHUB_SECRET"

# upgrade the deployment
$ helm upgrade drone \
  --reuse-values \
  --set persistence.enabled=false \
  --set server.env.DRONE=ORGS="" \
  --set server.env.DRONE_ADMIN="YOUR_GITHUB_USERNAME" \
  --set server.env.DRONE_OPEN=false \
  --set server.env.DRONE_GITHUB_CLIENT='YOUR_NEW_GITHUB_CLIENT'
   stable/drone
```

this way you have restricted the access to Drone to the only admin (you) and disabled persistence.

You can find these settings both in the [Drone Helm chart documentation](https://github.com/helm/charts/tree/master/stable/drone), in [Drone's documentation](http://readme.drone.io/) and in [this post](https://akomljen.com/set-up-a-drone-ci-cd-pipeline-with-kubernetes/).

## More information
We have written a [Medium post](https://medium.com/@lorenzo.soligo/setting-up-a-ci-cd-pipeline-with-drone-on-a-kubernetes-cluster-fc6779798430) about this project. Check it out for more information!

Also, the process of research and decision-making behind this project can be found [here](https://docs.google.com/document/d/1Xi3MglejhG_tBD4qmx8wAqZ77c7OptrqHgRq4H7K878/edit?usp=sharing).
