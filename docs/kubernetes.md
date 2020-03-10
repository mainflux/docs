Mainflux can be easily deployed on Kubernetes platform by using Helm Chart from offical [Mainflux DevOps GitHub repository](https://github.com/mainflux/devops).

## Prerequisites

- Kubernetes - install it or have access to a cluster
- kubectl - locally installed and configured
- Helm v3
- Stable Helm repository installed
- Nginx Ingress Controller

Kubectl is official Kubernetes command line client, follow [this instructions](https://kubernetes.io/docs/tasks/tools/install-kubectl/) to install it. 

Installing Helm v3 on Linux:
```
$ curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 > get_helm.sh
$ chmod 700 get_helm.sh
$ ./get_helm.sh
```

Verify it's installed:
```
$ helm version
```

Add stable chart repository:
```
helm repo add stable https://kubernetes-charts.storage.googleapis.com/
```

If Nginx Ingress Controller isn't already installed, this would install it in `mf` namespace:
```
helm install nginx-mainflux stable/nginx-ingress -n mf
```

## Deploying Mainflux

Get Helm charts from [Mainflux DevOps GitHub repository](https://github.com/mainflux/devops):
```
git clone https://github.com/mainflux/devops.git
cd devops/charts/mainflux
```

Update the on-disk dependencies to mirror Chart.yaml:

```
helm dependency update
```

Deploying release named `mainflux` in namespace named `mf` is done with just:
```
helm install mainflux -n mf
```

Mainflux is now deployed on your Kubernetes.

### Customizing installation


You can override default values while installing with `--set` option. For example, if you want to specify ingress hostname and pull `latest` tag of `users` image:
```
helm install mainflux -n mf --set ingress.hostname='example.com' --set users.image.tag='latest'
```

Or if release is already installed, you can update it:
```
helm upgrade mainflux -n mf --set ingress.hostname='example.com' --set users.image.tag='latest'
```

The following table lists the configurable parameters and their default values.

| Parameter                              | Description                                                                | Default        |
| -------------------------------------- | -------------------------------------------------------------------------- | -------------- |
| `defaults.logLevel`                    | Log level                                                                  | `debug`        |
| `defaults.image.pullPolicy`            | Docker Image Pull Policy                                                   | `IfNotPresent` |
| `defaults.image.repository`            | Docker Image Repository                                                    | `mainflux`     |
| `defaults.image.tag`                   | Docker Image Tag                                                           | `0.10.0`        |
| `nginx_internal.mtls.tls`              | TLS secret which contains the server cert/key                              | `''`           |
| `nginx_internal.mtls.intermediate_crt` | Generic secret which contains the intermediate cert used to verify clients | `''`           |
| `ingress.enabled`                      | Should the Nginx Ingress be created                                        | `true`         |
| `ingress.hostname`                     | Hostname for the Nginx Ingress                                             | `''`           |
| `ingress.tls.hostname`                 | Hostname of the Nginx Ingress certificate                                  | `''`           |
| `ingress.tls.secret`                   | TLS secret for the Nginx Ingress                                           | `''`           |

All Mainflux services can have their `logLevel`, `image.pullPolicy`, `image.repository` and `image.tag` overridden. The names of the services are:

- adapter_coap
- adapter_http
- mqtt.broker
- mqtt.proxy
- adapter_ws
- things
- ui
- users
- authn
- influxdb_writer
- infuxdb_reader