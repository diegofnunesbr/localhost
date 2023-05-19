# Cluster Localhost

## TL;DR

[Install ArgoCD](https://gitlab.miisy.me/devops/helm/core/argo-cd)

Install the Cluster Helm Chart
```console
helm install localhost-cluster .
```

## Introduction

This is the [ArgoCD Cluster](https://argo-cd.readthedocs.io/en/stable/operator-manual/declarative-setup/#clusters) Application that handles all resources that should be deployed to the `localhost` Cluster

A Cluster Application is defined as an [ArgoCD App of Apps](https://argo-cd.readthedocs.io/en/stable/operator-manual/cluster-bootstrapping/)

### ArgoCD Application Tree

This Helm Chart is deployed by ArgoCD using the following structure:
```
┌───────────────┐                 ┌───────────────┐                 ┌───────────────┐
│               │                 │               │                 │               │
│    Cluster    │                 │  App Of Apps  │                 │     Helm      │
│  Application  │◄────────────────│  Application  │◄────────────────│     Chart     │
│               │                 │               │                 │               │
└───────────────┘                 └───────────────┘                 └───────────────┘
my-cluster/                       my-apps/                          my-app/
├─ values.yaml                    ├─ values.yaml                    ├─ values.yaml 
├─ values/                        ├─ values.<env>.yaml              ├─ values.<env>.yaml
│ ├─ my-apps/                     ├─ values/                        ├─ values-version.<env>.yaml
│ │ ├─ values.yaml                │ ├─ my-app/
│ │ ├─ values.<env>.yaml          │ │ ├─ values.yaml
                                  │ │ ├─ values.<env>.yaml
                                  │ │ ├─ values-version.<env>.yaml
```

[Cluster Application](https://gitlab.miisy.me/devops/argocd/clusters/localhost): This Chart<br />
[App Of Apps Application](https://argo-cd.readthedocs.io/en/stable/operator-manual/cluster-bootstrapping/): Handles an Application that has other Applications defined<br />
Helm Chart: The Application Helm Chart Repository<br />

### Helm values hierarchy

See [Configuration](README.md#configuration)

## Prerequisites

- Kubernetes 1.12+
- Helm 3.1.0
- [ArgoCD](https://gitlab.miisy.me/devops/helm/core/argo-cd)

## Installing the Chart

This chart is installed with `helm` and it'll be handled by [ArgoCD](https://gitlab.miisy.me/devops/helm/core/argo-cd)

Install the Chart
```console
helm install localhost-cluster .
```

## Parameters

### Cluster

| Name                              | Description                                            | Value                            |
| --------------------------------- | ------------------------------------------------------ | -------------------------------- |
| `cluster.argoServer`              | URL that has the ArgoCD application deployed           | `https://kubernetes.default.svc` |
| `cluster.spec.destination.server` | Cluster that will have all applications deployed to    | `https://kubernetes.default.svc` |
| `cluster.self.name`               | Cluster name that will be prefixed to all applications | `localhost`                      |


### Core

| Name   | Description                  | Value  |
| ------ | ---------------------------- | ------ |
| `core` | Core Applications parameters | `true` |


## Configuration

To set specific cluster values that'll override the `App Of Apps` and `Application` values, add your `values.yaml` or `values.<env>.yaml` inside the `values/<app-name>` folder

The following example shows how the overriding of the `values.yaml` occurs:

#### Application "my-app" values
The file `values.yaml` contains all the default values used at the Helm Chart
```values.yaml
name: my-app
version: 1.0

drink: 
  type: coffee
  flavor: hazelnut
  ice: false
  sugar: false
  
food:
  type: sandwich
  flavor: ham
```
The file `values.trunk.yaml` defines the values that should override the default, for the `trunk`environment
```values.trunk.yaml
name: my-app-trunk
drink:   
  flavor: vanilla
```
The file `values-version.trunk.yaml` defines the version that should be deployed at the `trunk` environment
```values-version.trunk.yaml
version: 2.0
```

#### App Of Apps "my-apps" values
The file `values.yaml` contains all the default values that'll override the `my-app` values.<br />
The `override` object, is a special object that is used to take precedence over the `my-cluster` values
```values.yaml
my-app:
  drink:
    type: juice
    flavor: strawberry

override:
  food:
    type: cookie
    flavor: coconut
```
The file `values.trunk.yaml` defines the values that should override the default, for the `trunk` environment<br />
The `override` object can also be overridden for a specific environment
```values.trunk.yaml
my-app:
  drink:
    flavor: orange

override:
  food:    
    flavor: chocolate
```
The `/values/my-app/` folder, follows the same principle as the [Application "my-app" values](README.md#application-my-app-values)
You can use either or both ways to override the values. The files inside `/values/my-app/` override the ones at the root folder

`/values/my-app/values.yaml`
```/values/my-app/values.yaml
drink:
  type: juice
  flavor: strawberry
```
`/values/my-app/values.trunk.yaml`
```/values/my-app/values.trunk.yaml
drink:
  flavor: orange
```
#### Cluster "my-cluster" values
The file `values/my-apps/values.yaml` contains all the default values that'll override the `my-apps` values.<br />
```values/my-apps/values.yaml
drink:
  ice: true
  sugar: 1 spoon
  
food:
  type: pizza
  flavor: cheese
```
The file `values/my-apps/values.trunk.yaml` defines the values that should override the default, for the `trunk` environment
```values/my-apps/values.trunk.yaml
drink:    
  sugar: 2 spoons
  
food:
  type: pizza
  flavor: sausage
```

#### Rendered values
The file `values.yaml` takes the values according to the precedence explained and renders the following to be used at the deployment
```values.yaml
name: my-app-trunk
version: 2.0

drink: 
    type: juice
    flavor: orange
    ice: true
    sugar: 2 spoons
    
food:
  type: cookie  
  flavor: chocolate    
```
