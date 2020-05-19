![build-status][build-status]

Distributed load testing using kubernetes and locust
======================

The repository contains everything needed to run a distributed load testing environment in kubernetes using a locust master and locust slaves. 


## Table of contents
1. [Project structure](#project-structure)
2. [Prerequisites](#prerequisites)
3. [Running](#running)
    1. [tl;dr](#tldr)
    2. [Detailed installation](#detailed-installation)
        1. [Preparing the local kubernetes cluster](#preparing-the-local-kubernetes-cluster)
        2. [Building the docker image](#building-the-docker-image)
        3. [Installing the helm charts](#installing-the-helm-charts)
        4. [Confirm the installation and access locust dashboard](#confirm-the-installation-and-access-locust-dashboard)
4. [Architecture](#architecture)

## Project structure

    .
    ├── .vscode                                 # launch.json and tasks.json needed to debug the tasks.py in vsc
    ├── docker-image                            # The shared docker image for locust masters and slaves
    │   ├── locust-tasks                        # Python source code
    |   |   ├── requirements.txt                # Python dependencies for tasks.py
    |   |   ├── tasks.py                        # Locust tasks
    |   ├── run.sh                              # Shell script to determine if the docker containers should be master or slave
    │   ├── Dockerfile                          # Dockerfile
    ├── loadtest-chart                          # Helm chart
    |   ├── templates                           # Helm templates
    |   |   ├── _helpers.tpl                    # Helm helpers
    |   |   ├── locust-master-deployment.yaml   # Kubernetes deployment configuration for locust master
    |   |   ├── locust-master-service.yaml      # Kubernetes service configuration for locust master
    |   |   ├── locust-slave-deployment.yaml    # Kubernetes deployment configuration for locust slaves
    |   ├── Chart.yaml                          # Chart definition
    |   ├── values.yaml                         # Chart definition
    └── ...
___

## Prerequisites

| Product            |              Version         |                           Link                          | 
| :------------------|:-----------------------------|:--------------------------------------------------------|
| Python             | 2.7.15                       | [Windows][Python-Windows], [MacOS][Python-macOS]        |
| Docker             | 18.03.0-ce (23751)           | [Windows][Docker-Windows], [MacOS][Docker-macOS]        |
| kubectl            | 2.0.0                        | [Windows][kubectl-Windows], [MacOS][kubectl-MacOS]      |
| Minikube           | 0.27.0                       | [Windows][Minikube], [MacOS][Minikube]                  |
| helm               | 2.9.1                        | [Windows][helm-Windows] [MacOS][helm-macOS]             |


## Running

### Deploy locus project

```
helm install directory-locust-kubernetes/loadtest-chart/ --namespace locust --name locust --set hostAliases.ip=${web_ip}
```

### Start swarm by command lines

Locally :
```
curl -X POST -F 'locust_count=500' -F 'hatch_rate=5' http://<locust-url>:<locust-port>/swarm
```

In kubernetes : 
```
kubectl run --generator=run-pod/v1 startlocust --image=djbingham/curl --restart='OnFailure' -i --tty --rm --command -- curl -X POST -F 'locust_count=500' -F 'hatch_rate=10' http://locust-master.locust.svc.cluster.local:8089/swarm
```


### Stop load test by command lines

Locally: 
```
curl http://<locust-url>:<locust-port>/stop
```
In kubernetes : 
```
kubectl run --generator=run-pod/v1 stoplocuts --generator=run-pod/v1 --image=djbingham/curl --restart='OnFailure' -i --tty --rm --command -- curl http://locust-master.locust.svc.cluster.local:8089/stop
```



---

### Detailed installation

#### Building the docker image
In the root of the repo, run `docker build docker-image -t locust-tasks:latest`


![locust][locust]


### Architecture

[Python-Windows]: https://www.python.org/downloads/windows/
[Python-MacOS]: https://www.python.org/downloads/mac-osx/
[Docker-Windows]: https://docs.docker.com/docker-for-windows/install/#download-docker-for-windows
[Docker-MacOS]: https://docs.docker.com/docker-for-windows/install/#download-docker-for-windows
[kubectl-Windows]: https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-with-chocolatey-on-windows
[kubectl-MacOS]: https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-with-homebrew-on-macos
[Minikube]: https://kubernetes.io/docs/tasks/tools/install-minikube/ 
[helm-Windows]: https://docs.helm.sh/using_helm/#from-chocolatey-windows
[helm-macOS]: https://docs.helm.sh/using_helm/#from-homebrew-macos

[locust]: images/locust.png

[build-status]: https://travis-ci.org/joakimhew/locust-kubernetes.svg?branch=master