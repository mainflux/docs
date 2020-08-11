Mainflux can be easily deployed on Kubernetes platform by using Helm Chart from offical [Mainflux DevOps GitHub repository](https://github.com/mainflux/devops).

## Prerequisites

- Kubernetes - install it or have access to a cluster
- kubectl - locally installed and configured
- Helm v3
- Stable Helm repository installed
- Nginx Ingress Controller

Kubectl is official Kubernetes command line client, follow [this instructions](https://kubernetes.io/docs/tasks/tools/install-kubectl/) to install it.

Regaqrding the cluster control with `kubectl`, defualt config `.yaml` file should be `~/.kube/config`, so if you have another one (for example one downloaded from DO), it should be renamed.

Installing Helm v3 on Linux:
```bash
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 > get_helm.sh
chmod 700 get_helm.sh
./get_helm.sh
```

Verify it's installed:
```bash
helm version
```

Add stable chart repository:
```bash
helm repo add stable https://kubernetes-charts.storage.googleapis.com/
```

## Deploying Mainflux

Get Helm charts from [Mainflux DevOps GitHub repository](https://github.com/mainflux/devops):
```bash
git clone https://github.com/mainflux/devops.git
cd devops/charts/mainflux
```

Update the on-disk dependencies to mirror Chart.yaml:
```bash
helm dependency update
```

If you didn't already have namespace created you should do it with:
```bash
kubectl create namespace mf
```

Deploying release named `mainflux` in namespace named `mf` is done with just:
```bash
helm install mainflux . -n mf
```

Mainflux is now deployed on your Kubernetes.

If Nginx Ingress Controller isn't already installed, this would install it in `mf` namespace:
```bash
helm install nginx-mainflux stable/nginx-ingress -n mf
```

### Customizing installation


You can override default values while installing with `--set` option. For example, if you want to specify ingress hostname and pull `latest` tag of `users` image:
```bash
helm install mainflux -n mf --set ingress.hostname='example.com' --set users.image.tag='latest'
```

Or if release is already installed, you can update it:
```bash
helm upgrade mainflux -n mf --set ingress.hostname='example.com' --set users.image.tag='latest'
```

The following table lists the configurable parameters and their default values.

| Parameter                            | Description                                                                | Default      |
| ------------------------------------ | -------------------------------------------------------------------------- | ------------ |
| defaults.logLevel                    | Log level                                                                  | debug        |
| defaults.image.pullPolicy            | Docker Image Pull Policy                                                   | IfNotPresent |
| defaults.image.repository            | Docker Image Repository                                                    | mainflux     |
| defaults.image.tag                   | Docker Image Tag                                                           | 0.11.0       |
| defaults.replicaCount                | Replicas of MQTT adapter, Things, Envoy and Authn                          | 3            |
| nginx_internal.mtls.tls              | TLS secret which contains the server cert/key                              |              |
| nginx_internal.mtls.intermediate_crt | Generic secret which contains the intermediate cert used to verify clients |              |
| ingress.enabled                      | Should the Nginx Ingress be created                                        | true         |
| ingress.hostname                     | Hostname for the Nginx Ingress                                             |              |
| ingress.tls.hostname                 | Hostname of the Nginx Ingress certificate                                  |              |
| ingress.tls.secret                   | TLS secret for the Nginx Ingress                                           |              |
| nats.maxPayload                      | Maximum payload size in bytes that the NATS server will accept             | 268435456    |
| nats.replicaCount                    | NATS replicas                                                              | 3            |
| mqtt.broker.persistentVolume.size    | data Persistent Volume size                                                | 5Gi          |
| influxdb.enabled                     | Enable influxdb reader & writer                                            | false        |
| bootstrap.enabled                    | Enable bootstrap service                                                   | false        |
| adapter_opcua.enabled                | Enable OPC-UA Adapter                                                      | false        |
| twins.enabled                        | Enable twins service                                                       | false        |

All Mainflux services (both core and add-ons) can have their `logLevel`, `image.pullPolicy`, `image.repository` and `image.tag` overridden. 

Mainflux Core is a minimalistic set of required Mainflux services. They are all installed by default:

- adapter_coap
- adapter_http
- adapter_mqtt
- adapter_ws
- things
- ui
- users
- authn

Mainflux Add-ons are optional services that are disabled by default. Find in Configuration table paramaters for enabling them, i.e. to enable influxdb reader & writer you shoud run `helm install` with `--set influxdb=true`.
List of add-ons services in charts:

- bootstrap
- influxdb.writer
- influxdb.reader
- adapter_opcua
- twins

By default scale of MQTT adapter, Things, Envoy, Authn and NATS will be set to 3. It's recommended that you set this values to number of your nodes in Kubernetes cluster, i.e. `--set defaults.replicaCount=3 --set nats.replicaCount=3`

### Additional steps to configure Ingress Controller

To send MQTT messages to your host on ports `1883` and `8883` some additional steps are required in configuring NGINX Ingress Controler.

NGINX Ingress Controller uses ConfigMap to expose TCP services. That ConfigMap is included in helm chart in [ingress.yaml](https://github.com/mainflux/devops/blob/master/charts/mainflux/templates/ingress.yaml#L141) file assuming that location of ConfigMap should be `ingress-nginx/tcp-services`. If Ingress Controller expects it in some other namespace or with other name you should edit metadata in [ingress.yaml](https://github.com/mainflux/devops/blob/master/charts/mainflux/templates/ingress.yaml#L147). This location was set with `--tcp-services-configmap` flag and you can check it in deployment of Ingress Controller or add it there in [args section for nginx-ingress-controller](https://kubernetes.github.io/ingress-nginx/user-guide/cli-arguments/) if it's not already specified. This is explained in [NGINX Ingress documentation](https://kubernetes.github.io/ingress-nginx/user-guide/exposing-tcp-udp-services/)

Also, those two ports need to be exposed in the Service defined for the Ingress. You can do that with command that edit your service:

```kubectl edit svc -n ingress-nginx nginx-ingress-ingress-nginx-controller```

and add in spec->ports:

```
  - name: mqtt
    port: 1883
    protocol: TCP
    targetPort: 1883
  - name: mqtts
    port: 8883
    protocol: TCP
    targetPort: 8883
```
