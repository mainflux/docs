Mainflux can be easily deployed on Kubernetes platform by using Helm Chart from official [Mainflux DevOps GitHub repository](https://github.com/mainflux/devops).

## Prerequisites
- Kubernetes
- kubectl
- Helm v3
- Stable Helm repository
- Nginx Ingress Controller

### Kubernetes
Kubernetes is an open source container orchestration engine for automating deployment, scaling, and management of containerised applications.
Install it locally or have access to a cluster. Follow [this instructions](https://kubernetes.io/docs/setup/) if you need more information. 

### kubectl
Kubectl is official Kubernetes command line client. Follow [this instructions](https://kubernetes.io/docs/tasks/tools/install-kubectl/) to install it.

Regarding the cluster control with `kubectl`, default config `.yaml` file should be `~/.kube/config`, so if you have another one (for example one downloaded from DO), it should be renamed.

### Helm v3
Helm is the package manager for Kubernetes. Follow [this instructions](https://helm.sh/docs/intro/install/) to install it.

### Stable Helm repository
Add a stable chart repository:
```bash
helm repo add stable https://kubernetes-charts.storage.googleapis.com/
```

### Nginx Ingress Conntroller
Follow [this instructions](https://kubernetes.github.io/ingress-nginx/deploy/) to install it.

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
| defaults.natsPort                    | NATS port                                                                  | 4222         |
| defaults.jaegerPort                  | Jaeger port                                                                | 6831         |
| nginx_internal.mtls.tls              | TLS secret which contains the server cert/key                              |              |
| nginx_internal.mtls.intermediate_crt | Generic secret which contains the intermediate cert used to verify clients |              |
| ingress.enabled                      | Should the Nginx Ingress be created                                        | true         |
| ingress.hostname                     | Hostname for the Nginx Ingress                                             |              |
| ingress.tls.hostname                 | Hostname of the Nginx Ingress certificate                                  |              |
| ingress.tls.secret                   | TLS secret for the Nginx Ingress                                           |              |
| nats.maxPayload                      | Maximum payload size in bytes that the NATS server will accept             | 268435456    |
| nats.replicaCount                    | NATS replicas                                                              | 3            |
| authn.dbPort                         | AuthN service DB port                                                      | 5432         |
| authn.grpcPort                       | AuthN service gRPC port                                                    | 8181         |
| authn.httpPort                       | AuthN service HTTP port                                                    | 8189         |
| users.dbPort                         | Users service DB port                                                      | 5432         |
| users.httpPort                       | Users service HTTP port                                                    | 8180         |
| things.dbPort                        | Things service DB port                                                     | 5432         |
| things.httpPort                      | Things service HTTP port                                                   | 8182         |
| things.authGrpcPort                  | Things service Auth gRPC port                                              | 8183         |
| things.authHttpPort                  | Things service Auth HTTP port                                              | 8989         |
| things.redisESPort                   | Things service Redis Event Store port                                      | 6379         |
| things.redisCachePort                | Things service Redis Auth Cache port                                       | 6379         |
| adapter_http.httpPort                | HTTP adapter port                                                          | 8185         |
| mqtt.proxy.mqttPort                  | MQTT adapter proxy port                                                    | 1884         |
| mqtt.proxy.wsPort                    | MQTT adapter proxy WS port                                                 | 8081         |
| mqtt.broker.mqttPort                 | MQTT adapter broker port                                                   | 1883         |
| mqtt.broker.wsPort                   | MQTT adapter broker WS port                                                | 8080         |
| mqtt.broker.persistentVolume.size    | MQTT adapter broker data Persistent Volume size                            | 5Gi          |
| mqtt.redisESPort                     | MQTT adapter Event Store port                                              | 6379         |
| mqtt.redisCachePort                  | MQTT adapter Redis Auth Cache port                                         | 6379         |
| adapter_coap.udpPort                 | CoAP adapter UDP port                                                      | 5683         |
| ui.port                              | UI port                                                                    | 3000         |
| bootstrap.enabled                    | Enable bootstrap service                                                   | false        |
| bootstrap.dbPort                     | Bootstrap service DB port                                                  | 5432         |
| bootstrap.httpPort                   | Bootstrap service HTTP port                                                | 8182         |
| bootstrap.redisESPort                | Bootstrap service Redis Event Store port                                   | 6379         |
| influxdb.enabled                     | Enable InfluxDB reader & writer                                            | false        |
| influxdb.dbPort                      | InfluxDB port                                                              | 8086         |
| influxdb.writer.httpPort             | InfluxDB writer HTTP port                                                  | 8900         |
| influxdb.reader.httpPort             | InfluxDB reader HTTP port                                                  | 8905         |
| adapter_opcua.enabled                | Enable OPC-UA adapter                                                      | false        |
| adapter_opcua.httpPort               | OPC-UA adapter HTTP port                                                   | 8188         |
| adapter_opcua.redisRouteMapPort      | OPC-UA adapter Redis Auth Cache port                                       | 6379         |
| adapter_lora.enabled                 | Enable LoRa adapter                                                        | false        |
| adapter_lora.httpPort                | LoRa adapter HTTP port                                                     | 8187         |
| adapter_lora.redisRouteMapPort       | LoRa adapter Redis Auth Cache port                                         | 6379         |
| twins.enabled                        | Enable twins service                                                       | false        |
| twins.dbPort                         | Twins service DB port                                                      | 27017        |
| twins.httpPort                       | Twins service HTTP port                                                    | 9021         |
| twins.redisCachePort                 | Twins service Redis Cache port                                             | 6379         |

All Mainflux services (both core and add-ons) can have their `logLevel`, `image.pullPolicy`, `image.repository` and `image.tag` overridden.

Mainflux Core is a minimalistic set of required Mainflux services. They are all installed by default:

- authn
- users
- things
- adapter_http
- adapter_mqtt
- adapter_coap
- ui

Mainflux Add-ons are optional services that are disabled by default. Find in Configuration table parameters for enabling them, i.e. to enable influxdb reader & writer you should run `helm install` with `--set influxdb=true`.
List of add-ons services in charts:

- bootstrap
- influxdb.writer
- influxdb.reader
- adapter_opcua
- adapter_lora
- twins

By default scale of MQTT adapter, Things, Envoy, Authn and NATS will be set to 3. It's recommended that you set this values to number of your nodes in Kubernetes cluster, i.e. `--set defaults.replicaCount=3 --set nats.replicaCount=3`

### Additional steps to configure Ingress Controller
To send MQTT messages to your host on ports `1883` and `8883` some additional steps are required in configuring NGINX Ingress Controller.

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

# TLS & mTLS

For testing purposes you can generate certificates as explained in detail in (authentication)[/authentication] chapter of this document. So, you can use (this script)[https://github.com/mainflux/mainflux/blob/master/docker/ssl/Makefile] and after replacing all `localhost` with your hostname, run:

```
make ca
make server_cert
make thing_cert KEY=<thing_key>
```

you should get in `certs` folder these certificates that we will use for setting up TLS and mTLS:

```
ca.crt
ca.key
ca.srl
mainflux-server.crt
mainflux-server.key
thing.crt
thing.key
```

Create kubernetes secrets using those certificates with running commands from (secrets script)[https://github.com/mainflux/devops/blob/master/charts/mainflux/secrets/secrets.sh]. In this example secrets are created in `mf` namespace:

```
kubectl -n mf create secret tls mainflux-server \
    --key mainflux-server.key \
    --cert mainflux-server.crt

kubectl -n mf create secret generic ca \
    --from-file=ca.crt
```

 You can check if they are succesfully created:
 ```
 kubectl get secrets -n mf
 ```

 And now set ingress.hostname, ingress.tls.hostname to your hostname and ingress.tls.secret to `mainflux-server` and after helm update you have secured ingress with TLS certificate.

For mTLS you need to set `nginx_internal.mtls.tls="mainflux-server"` and `nginx_internal.mtls.intermediate_crt="ca"`.

Now you can test sending mqtt message with this parameters:

```
mosquitto_pub -d -L mqtts://<thing_id>:<thing_key>@example.com:8883/channels/<channel_id>/messages  --cert  thing.crt --key thing.key --cafile ca.crt  -m "test-message"
```
