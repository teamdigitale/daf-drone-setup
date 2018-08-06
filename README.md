# drone-poc

A Drone.io *proof-of-concept* for the Team Digitale

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


## Installation and Setup

### Kubernetes Requirements

In general to configure drone in kubernetes we need:
- an ingress entry;
- a Persistent Volume and a Persistent Volume Claim for data and configurations persistence.

Detailed instructions can be found [here](kubernetes/README.md).


### Drone Configurations
A tutorial for this step can be found [here](drone-install/README.md).


### Configuring the pipelines
Both the instructions and examples can be found in [drone-pipeline](drone-pipeline/README.md)

- Test `.drone.yml`: ![here](drone-pipeline/test/.drone.yml). This pipeline runs tests and sends Slack notifications about them.

- Build & deploy `.drone.yml`: ![here](drone-pipeline/build/.drone.yml). This pipeline builds the service's Docker image and updates the Kubernetes deployment to match the newly created version.
