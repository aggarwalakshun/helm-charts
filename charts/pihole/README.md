# Helm chart for Pihole on kubernetes

## Table of contents
- [Adding helm repo](#add-helm-repo)
- [View supported helm values](#print-possible-values)
- [Installation](#installation)
- [Port 53 is in use issue](#troubleshooting)

## Add helm repo
-   ```
    helm repo add helm-charts https://aggarwalakshun.github.io/helm-charts && helm repo update
    ```

## Print possible values
-   ```
    helm show values helm-charts/pihole
    ```
- You can download a sample [`values.yaml`](/charts/pihole/values.yaml) from this repo

| Parameter | Description | Default |
| :-------- | :---------: | ------: |
| `.Values.name` | (Optional) Can be anything | pihole |
| `.Values.namespace` | (Optional) Target Namespace | default |
| `.Values.replicas` | (Optional) Number of replicas | 1 |
| `.Values.versionOverride` | (Optional) Override image tag | Defaults to appVersion from [Chart.yaml](/charts/pihole/Chart.yaml) |
| `.Values.claimName` | (Required) PVC for pihole data | Required |
| `.Values.TZ` | (Optional) Timezone for pihole | UTC |
| `.Values.FTLCONF_LOCAL_IPV4` | (Optional) Upstream DNS server(s) for Pi-hole to forward queries to, separated by a semicolon | 8.8.8.8;8.8.4.4 |
| `.Values.TAIL_FTL_LOG` | (Optional) Whether or not to output the FTL log when running the container. Can be disabled by setting the value to 0 | 1 |
| `.Values.piholeUID` | (Optional) Overrides image's default pihole user id to match a host user id | 1000 |
| `.Values.piholeGID` | (Optional) Overrides image's default pihole group id to match a host group id | 1000 |
| `.Values.serviceType` | (Optional) Service type for pihole dashboard | ClusterIP |
| `.Values.servicePort` | (Optional) Service port for pihole dashboard | 8080 |
| `.Values.serviceNodePort` | (Optional) NodePort for pihole dashboard | 30080 |

## Installation
### Pre install checks
1. Ensure existence of PVC for pihole data.
2. Ensure reference of PVC name in `values.yaml`.

### Install with helm
-   ```
    helm install pihole helm-charts/pihole --values values.yaml
    ```
### Install from repo
-   ```
    git clone https://github.com/aggarwalakshun/helm-charts.git && cd helm-charts/charts
    ```
-   ```
    helm install pihole pihole/ --values pihole/values.yaml
    ```

## Troubleshooting
- A lot of modern distros come with `systemd-resolved` pre-installed which uses port 53 required by pihole.
- To get around this, stub resolver needs to disabled using the commands -
```
# disable stub resolver
sudo sed -r -i.orig 's/#?DNSStubListener=yes/DNSStubListener=no/g' /etc/systemd/resolved.conf

# remove symlink
sudo sh -c 'rm /etc/resolv.conf && ln -s /run/systemd/resolve/resolv.conf /etc/resolv.conf'

# restart systemd-resolved
sudo systemctl restart systemd-resolved
```

- Restart pihole after executing these commands by restarting pihole deployment
```
kubectl rollout restart deployment <deployment_name>
```
