

## Drone Pipeline

### First pipeline: Test

We will configure some automatic tests using [this](https://github.com/teamdigitale/daf-srv-storage) repository. Tests are run by executing `sbt test`.

We simply need to create a *test* pipeline in order to run this test and check whether they run successfully.

First of all, let's activate the repository in Drone:

![img](https://i.imgur.com/VBvY5UB.png)

And let's add  [the .drone.yml file](test/.drone.yml) to the repository

As we can see, the configuration is extremely simple and relies on a [Docker image](https://hub.docker.com/r/hseeberger/scala-sbt/) containing Scala and SBT.

Once the file is in the repository, every commit will trigger the test pipeline, and the results will be clear in Drone's web-UI

![img](https://i.imgur.com/bIWlCNY.png)


Of course, we can also read the logs to check every step that was taken

![img](https://i.imgur.com/Opp5jd7.png)

In the latest version of this pipeline, Slack notifications have been added. This will result in a Slack message sent every time a build passes/fails. For instance: ![](https://i.imgur.com/TJGs2lF.png)



### Build and Deploy

We will configure build & deploy with [this](https://github.com/teamdigitale/daf-models/tree/master/outbox-classification/web-api) repository, using the [test deployment file](https://github.com/teamdigitale/daf-models/blob/master/outbox-classification/web-api/kube-deploy-test.yml). The service is reachable within the test VPN @ http://ml-api.daf.teamdigitale.test/.

We will use the following [pipeline](build/.drone.yml), which relies on two secrets, `docker_username` and `docker_password`, which contain Nexus' username and password. These secrets can either be created from Drone's web UI (repo_name -> secrets) or from the Drone CLI.