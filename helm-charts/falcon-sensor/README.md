# CrowdStrike Falcon Helm Chart

[Falcon](https://www.crowdstrike.com/) is the [CrowdStrike](https://www.crowdstrike.com/)
platform purpose-built to stop breaches via a unified set of cloud-delivered
technologies that prevent all types of attacks — including malware and much
more.

# Kubernetes Cluster Compatability

The Falcon Helm chart has been tested to deploy on the following Kubernetes distributions:

* Amazon Elastic Kubernetes Service (EKS)
* Azure Kubernetes Service (AKS) - Linux Nodes Only
* Google Kubernetes Engine (GKE)
* Rancher K3s
* Red Hat OpenShift Container Platform 4.6+ - Falcon Container sensor only

# Dependencies

1. Requires a x86_64 Kubernetes cluster
1. Must be a CrowdStrike customer with access to the Falcon Linux Sensor (container image) and Falcon Container from the CrowdStrike Container Registry.
1. Kubernetes nodes must be Linux distributions supported by CrowdStrike.
1. Before deploying the Helm chart, you should have a Falcon Linux Sensor and/or Falcon Container sensor in your own container registry or use CrowdStrike's registry before installing the Helm Chart. See the Deployment Considerations for more.
1. Helm 3.x is installed and supported by the Kubernetes vendor.

# Installation

### Add the CrowdStrike Falcon Helm repository

```
helm repo add crowdstrike https://crowdstrike.github.io/falcon-helm
```

### Update the local Helm repository Cache

```
helm repo update
```

# Falcon Configuration Options

The following tables lists the Falcon Sensor configurable parameters and their default values.

| Parameter                   | Description                                          | Default                   |
|:----------------------------|:-----------------------------------------------------|:------------------------- |
| `falcon.cid`                | CrowdStrike Customer ID (CID)                        | None       (Required)     |
| `falcon.apd`                | Enable/Disable the Proxy.                            | None                      |
| `falcon.aph`                | App Proxy Hostname (APH)                             | None                      |
| `falcon.app`                | App Proxy Port (APP)                                 | None                      |
| `falcon.trace`              | Set trace level                                      | `none`                    |
| `falcon.feature`            | Sensor Feature options                               | None                      |
| `falcon.message_log`        | Enable message log (true/false)                      | None                      |
| `falcon.billing`            | Utilize default or metered billing                   | None                      |
| `falcon.tags`               | Comma separated list of tags for sensor grouping     | None                      |
| `falcon.provisioning_token` | Provisioning token value                             | None                      |


## Installing on Kubernetes Cluster Nodes

### Deployment Considerations

To ensure a successful deployment, you will want to ensure that:
1. By default, the Helm Chart installs in the `default` namespace. Best practices for deploying to Kubernetes is to create a new namespace. This can be done by adding `-n falcon-system --create-namespace` to your `helm install` command.
1. The Falcon Linux Sensor (not the Falcon Container) should be used as the container image to deploy to Kubernetes nodes.
1. You must be a cluster administrator to deploy Helm Charts to the cluster.
1. When deploying the Falcon Linux Sensor (container image) to Kubernetes nodes, it is a requirement that the Falcon Sensor run as a privileged container so that the Sensor can properly work with the kernel. This is a requirement for any kernel module that gets deployed to any container-optimized operating system regardless of whether it is a security sensor, graphics card driver, etc.
1. The Falcon Linux Sensor should be deployed to Kubernetes environments that allow node access or installation via a Kubernetes DaemonSet.
1. The Falcon Linux Sensor will create `/opt/CrowdStrike` on the Kubernetes nodes. DO NOT DELETE this folder.
1. CrowdStrike's Helm Chart is a project, not a product, and released to the community as a way to automate sensor deployment to kubernetes clusters. The upstream repository for this project is [https://github.com/CrowdStrike/falcon-helm](https://github.com/CrowdStrike/falcon-helm).

### Install CrowdStrike Falcon Helm Chart on Kubernetes Nodes

```
helm upgrade --install falcon-helm crowdstrike/falcon-sensor \
    --set falcon.cid="<CrowdStrike_CID>" \
    --set node.image.repository="<Your_Registry>/falcon-node-sensor"
```

Above command will install the CrowdStrike Falcon Helm Chart with the release name `falcon-helm` in the namespace your `kubectl` context is currently set to.
You can install also install into a customized namespace by running the following:

```
helm upgrade --install falcon-helm crowdstrike/falcon-sensor \
    -n falcon-system --create-namespace \
    --set falcon.cid="<CrowdStrike_CID>" \
    --set node.image.repository="<Your_Registry>/falcon-node-sensor"
```

For more details please see the [falcon-helm](https://github.com/CrowdStrike/falcon-helm) repository.

### Node Configuration

The following tables lists the more common configurable parameters of the chart and their default values for installing on a Kubernetes node.

| Parameter                       | Description                                                          | Default                                   |
|:--------------------------------|:---------------------------------------------------------------------|:----------------------------------------- |
| `node.enabled`                  | Enable installation on the Kubernetes node                           | `true`                                    |
| `node.image.repository`         | Falcon Sensor Node registry/image name                               | `falcon-node-sensor`                      |
| `node.image.tag`                | The version of the official image to use                             | `latest`                                  |
| `node.image.pullPolicy`         | Policy for updating images                                           | `Always`                                  |
| `node.image.pullSecrets`        | Pull secrets for private registry                                    | `{}`                                      |
| `falcon.cid`                    | CrowdStrike Customer ID (CID)                                        | None       (Required)                     |

`falcon.cid` and `node.image.repository` are required values.

## Installing in Kubernetes Cluster as a Sidecar

### Deployment Considerations

To ensure a successful deployment, you will want to ensure that:
1. You must be a cluster administrator to deploy Helm Charts to the cluster.
1. When deploying the Falcon Container as a sidecar sensor, make sure that there are no firewall rules blocking communication to the Mutating Webhook. This will most likely result in a `context deadline exceeded` error. The default port for the Webhook is `4433`.
1. The Falcon Container as a sidecar sensor should be deployed to Kubernetes managed environments, or environments that do not allow node access or installation via a Kubernetes DaemonSet.
1. CrowdStrike's Helm Chart is a project, not a product, and released to the community as a way to automate sensor deployment to kubernetes clusters. The upstream repository for this project is [https://github.com/CrowdStrike/falcon-helm](https://github.com/CrowdStrike/falcon-helm).

### Install CrowdStrike Falcon Helm Chart in Kubernetes Cluster as a Sidecar

```
helm upgrade --install falcon-helm crowdstrike/falcon-sensor \
    --set node.enabled=false
    --set container.enabled=true \
    --set falcon.cid="<CrowdStrike_CID>" \
    --set container.image.repository="<Your_Registry>/falcon-sensor"
```

Above command will install the CrowdStrike Falcon Helm Chart with the release name `falcon-helm` in the namespace your `kubectl` context is currently set to.
You can install also install into a customized namespace by running the following:

```
helm upgrade --install falcon-helm crowdstrike/falcon-sensor \
    -n falcon-system --create-namespace \
    --set node.enabled=false
    --set container.enabled=true \
    --set falcon.cid="<CrowdStrike_CID>" \
    --set container.image.repository="<Your_Registry>/falcon-sensor"
```

### Container Sensor Configuration

The following tables lists the more common configurable parameters of the chart and their default values for installing the Container sensor as a Sidecar.

| Parameter                                        | Description                                                             | Default                                   |
|:-------------------------------------------------|:------------------------------------------------------------------------|:----------------------------------------- |
| `container.enabled`                              | Enable installation on the Kubernetes node                              | `false`                                   |
| `container.disableNSInjection`                   | Disable injection for all Namespaces                                    | `false`                                   |
| `container.disablePodInjection`                  | Disable injection for all Pods                                          | `false`                                   |
| `container.certExpiration`                       | Certificate validity duration in number of days                         | `3650`                                    |
| `container.image.repository`                     | Falcon Sensor Node registry/image name                                  | `falcon-sensor`                           |
| `container.image.tag`                            | The version of the official image to use                                | `latest`                                  |
| `container.image.pullPolicy`                     | Policy for updating images                                              | `Always`                                  |
| `container.image.pullSecrets.enable`             | Enable pull secrets for private registry                                | `false`                                   |
| `container.image.pullSecrets.namespaces`         | Namespaces that should the Falcon sensor from an authenticated registry | None                                      |
| `container.image.pullSecrets.registryConfigJSON` | base64 encoded docker config json for the pull secret                   | None                                      |
| `falcon.cid`                                     | CrowdStrike Customer ID (CID)                                           | None       (Required)                     |

`falcon.cid` and `container.image.repository` are required values.

### Uninstall Helm Chart
To uninstall, run the following command:
```
helm uninstall falcon-helm
```

To uninstall from a custom namespace, run the following command:
```
helm uninstall falcon-helm -n falcon-system
```
