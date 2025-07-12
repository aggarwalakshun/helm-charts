# Helm Chart for immich on kubernetes

## Table of Contents
- [Adding helm repo](#add-helm-repo)
- [View all supported values.yaml values](#print-possible-values)
- [Installation](#installation)
- [Info regarding hardware acceleration](#additional-information)

## Add helm repo
- >`helm repo add helm-charts https://aggarwalakshun.github.io/helm-charts`
- >`helm repo update`

## Print possible values
- >`helm show values helm-charts/immich`
- You can download sample [`values.yaml`](/helm-charts/charts/immich/values.yaml) from this repo

| Parameter | Description | Default |
| :-------- | :---------: | ------: |
| `.Values.name` | (Required) Can be anything  | |
| `.Values.namespace` | (Required) Can be anything  | |
| `.Values.immichVersion` | (Required) [Check latest immich version](https://github.com/immich-app/immich/releases) | |
| `.Values.redisImage` | (Required) Check [docker_compose.yml](https://github.com/immich-app/immich/releases) of desired immich app version | |
| `.Values.postgresImage` |(Required) Check [docker_compose.yml](https://github.com/immich-app/immich/releases) of desired immich app version | |
| `.Values.replicas.app` | (Required) Number of replicas for main app | `1` |
| `.Values.replicas.ml` | (Required) Number of replicas for machine-learning deployment | `1` |
| `.Values.env.timezone` | (Optional) Timezone for immich server | `UTC` |
| `.Values.env.environment` | (Optional) Environment (`production`, `development`) | `production` |
| `.Values.env.logLevel` | (Optional) Can be one of `verbose`, `debug`, `log`, `warn`, `error` | `log` |
| `.Values.env.noColor` | (Optional) Set to `true` to disable color-coded log output | `false` |
| `.Values.env.processInvalidImages` | (Optional) When `true`, generate thumbnails for invalid images | `false` |
| `.Values.env.db.port` | (Optional) Don't change unless you know what you're doing. Port for postgresql database | `5432` |
| `.Values.env.db.username` | (Required) Postgresql database username | `postgres` |
| `.Values.env.db.name` | (Required) Postgresql database name | `immich` |
| `.Values.env.redisPort` | (Optional) Don't change unless you know what you're doing. Port for redis database | `6379` |
| `.Values.env.modelTTL` | (Optional) Inactivity time (s) before a model is unloaded (disabled if <= 0) | `300` |
| `.Values.env.modelTTLPollS` | (Optional) Interval (s) between checks for the model TTL (disabled if <= 0) | `60` |
| `.Values.env.interOpThreads` | (Optional) Number of parallel model operations | `1` |
| `.Values.env.intraOpThreads` | (Optional) Number of threads for each model operation | `2` |
| `.Values.env.workers` | (Optional) Number of worker processes to spawn | `1` |
| `.Values.env.httpKeepAliveTimeoutS` | (Optional) HTTP Keep-alive time in seconds | `2` |
| `.Values.hwAcceleration.enabled` | (Required) Enable  or disable hardware acceleration for machine learning (`true` or `false`) | |
| `.Values.hwAcceleration.type` | (Required if `.Values.hwAcceleration.enabled` = `true`) Can be either `intel` or `nvidia` | |
| `.Values.secrets.db.name` | (Required) Name of postgres database password secret | |
| `.Values.secrets.db.key` | (Required) Key of postgres database password secret | |
| `.Values.persistence.pictures.existingClaimName` | (Required) PVC for storing pictures | |
| `.Values.persistence.modelCache.existingClaimName` | (Required) PVC for machine learning model cache | |
| `.Values.persistence.postgres.existingClaimName` | (Required) PVC for postgres database | |
| `.Values.services.app.type` | (Required) Choose how to expose immich app. Can be one of `LoadBalancer`, `NodePort`, or `ClusterIP`| |
| `.Values.services.app.port` | (Required if app service type = `LoadBalancer`) | `2283` |
| `.Values.services.app.nodePort` | (Required if app service type = `NodePort`) | `32283` |
| `.Values.services.ml.type` | (Required) Choose how to expose immich-machine-learning deployment. Can be one of `LoadBalancer`, `NodePort`, or `ClusterIP`| |
| `.Values.services.ml.port` | (Required if machine learning service type = `LoadBalancer`) | `3003` |
| `.Values.services.ml.nodePort` | (Required if machine learning service type = `NodePort`) | `30003` |

## Installation
### Pre install checks
1. Make sure you have created PersistentVolumeClaims for pictures, machine-learning cache and postgres.
2. Make sure the PVCs are referred correctly in `values.yaml`

### Install with helm
- >`helm install immich helm-charts/immich --values values.yaml`
### Install from repo
- >`git clone https://github.com/aggarwalakshun/helm-charts.git`
- >`cd helm-charts/charts`
- >`helm install immich immich/ --values immich/values.yaml`

## Additional information
### Hardware acceleration using nvidia GPU
Ensure you have the nvidia gpu-operator installed and configured. For more information refer to [nvidia docs](https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/latest/index.html).

#### Summary of how to install the gpu-operator
1. Ensure you have installed nvidia-gpu-driver (proprietary) on host node.
2. >`kubectl create ns gpu-operator`
3. >`kubectl label --overwrite ns gpu-operator pod-security.kubernetes.io/enforce=privileged`
4. >`helm repo add nvidia https://helm.ngc.nvidia.com/nvidia && helm repo update`

- **For k8s**
    >`helm install --wait --generate-name -n gpu-operator nvidia/gpu-operator --set driver.enabled=false`
- **For k3s**
    ```
    helm install --wait --generate-name -n gpu-operator nvidia/gpu-operator \
    --set driver.enabled=false \
    --set toolkit.env[0].name=CONTAINERD_CONFIG \
    --set toolkit.env[0].value=/var/lib/rancher/k3s/agent/etc/containerd/config.toml \
    --set toolkit.env[1].name=CONTAINERD_SOCKET \
    --set toolkit.env[1].value=/run/k3s/containerd/containerd.sock
    ```

### Hardware acceleration using intel GPU
1. Tag nodes with intel GPUs.
    >`kubectl label nodes node-1 intel.feature.node.kubernetes.io/gpu=true`
2. Install a cert-manager.
    1. >`helm repo add jetstack https://charts.jetstack.io && helm repo update`
    2. >`helm install cert-manager jetstack/cert-manager --namespace cert-manager --create-namespace --set crds.enabled=true`
3. Install device plugin operator and GPU plugin.
    1. >`helm repo add intel https://intel.github.io/helm-charts && helm repo update`
    2. >`helm install device-plugin-operator intel/intel-device-plugins-operator`
    3. >`helm install gpu-device-plugin intel/intel-device-plugins-gpu`
