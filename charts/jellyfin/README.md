# Helm chart for Jellyfin on kubernetes

## Table of contents
- [Adding helm repo](#add-helm-repo)
- [View supported helm values](#print-possible-values)
- [Installation](#installation)
- [Info regarding hardware acceleration](#additional-information)

## Add helm repo
-   ```
    helm repo add helm-charts https://aggarwalakshun.github.io/helm-charts && helm repo update
    ```

## Print possible values
-   ```
    helm show values helm-charts/jellyfin
    ```
- You can download a sample [`values.yaml`](/charts/jellyfin/values.yaml) from this repo

| Parameter | Description | Default |
| :-------- | :---------: | ------: |
| `.Values.name` | (Optional) Can be anything  |jellyfin |
| `.Values.namespace` | (Optional) Target namespace | default |
| `.Values.replicas` | (Optional) Number of replicas | 1 |
| `.Values.versionOverride` | (Optional) Override image tag | Defaults to appVersion from [Chart.yaml](/charts/jellyfin/Chart.yaml) |
| `.Values.hwAccel.enabled` | (Required) Whether or not to enable GPU hardware acceleration. Requires additional [config](#additional-information) | Required |
| `.Values.hwAccel.type` | (Optional) GPU Vendor. Only `nvidia` or `intel` are supported | Defaults to CPU for transcoding |
| `.Values.persistence.configClaim` | (Required) Name of PVC for jellyfin's config data | Required |
| `.Values.persistence.cacheClaim` | (Required) Name of PVC for jellyfin cache | Required |
| `.Values.persistence.mediaClaim` | (Required) Name of PVC containing media library | Required |
| `.Values.serviceType` | (Optional) Type of service for jellyfin deployment | ClusterIP |
| `.Values.servicePort` | (Optional) Service Port | 8096 |
| `.Values.serviceNodePort` | (Optional) Service nodePort | 30096 |

## Installation
### Pre install checks
1. Ensure existence of PVCs for jellyfin cache, jellyfin config, and a PVC with your media library.
2. You may want to refer to an NFS share for the media library. You can do so with the following example.

- **pv.yml**
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: jellyfin-media-pv
  namespace: default
spec:
  capacity:
    storage: 200Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  nfs:
    path: "/media/"
    server: "10.80.80.147"
    readOnly: true
```
- **pvc.yml**
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: jellyfin-media-claim
  namespace: default
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 200Gi
  storageClassName: "" # Empty string must be explicitly set otherwise default StorageClass will be set
  volumeName: jellyfin-media-pv
```

### Install with helm
-   ```
    helm install jellyfin helm-charts/jellyfin --values values.yaml
    ```
### Install from repo
-   ```
    git clone https://github.com/aggarwalakshun/helm-charts.git && cd helm-charts/charts
    ```
-   ```
    helm install jellyfin jellyfin/ --values jellyfin/values.yaml
    ```

## Additional information
### Hardware acceleration using nvidia GPU
Ensure you have the nvidia gpu-operator installed and configured. For more information refer to [nvidia docs](https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/latest/index.html).

#### Summary of how to install the gpu-operator
1. Ensure you have installed nvidia-gpu-driver (proprietary) on host node.
2.  ```
    kubectl create ns gpu-operator
    ```
3.  ```
    kubectl label --overwrite ns gpu-operator pod-security.kubernetes.io/enforce=privileged
    ```
4.  ```
    helm repo add nvidia https://helm.ngc.nvidia.com/nvidia && helm repo update
    ```

- **For k8s**
    ```
    helm install --wait --generate-name -n gpu-operator nvidia/gpu-operator --set driver.enabled=false
    ```
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
    ```
    kubectl label nodes node-1 intel.feature.node.kubernetes.io/gpu=true
    ```
2. Install a cert-manager.
    1.  ```
        helm repo add jetstack https://charts.jetstack.io && helm repo update
        ```
    2.  ```
        helm install cert-manager jetstack/cert-manager --namespace cert-manager --create-namespace --set crds.enabled=true
        ```
3. Install device plugin operator and GPU plugin.
    1.  ```
        helm repo add intel https://intel.github.io/helm-charts && helm repo update
        ```
    2.  ```
        helm install device-plugin-operator intel/intel-device-plugins-operator
        ```
    3.  ```
        helm install gpu-device-plugin intel/intel-device-plugins-gpu
        ```
