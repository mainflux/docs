# Export
Mainflux Export service can send message from one Mainflux cloud to another via MQTT, or it can send messages from edge gateway to Mainflux Cloud.
Export service is subscribed to local message bus and connected to MQTT broker in the cloud.  
Messages collected on local message bus are redirected to the cloud.
When connection is lost, messages from the local bus are stored into `Redis` stream. Upon connection reestablishment `Export` service consumes messages from `Redis` stream and sends it to the Mainflux cloud.
Additonaly `Export` service publishes livelines status to `Agent` via NATS subject `heartbeat.export.service`


## Install
Get the code:

```bash
go get github.com/mainflux/export
cd $GOPATH/github.com/mainflux/export
```

Make:
```bash
make
```

## Usage

```bash
cd build
./mainflux-export
```

## Configuration
By default `Export` service looks for config file at [`../configs/config.toml`][conftoml] if no env vars are specified.  

```toml
[exp]
  log_level = "debug"
  nats = "localhost:4222"
  port = "8170"

[mqtt]
  username = "<thing_id>"
  password = "<thing_password>"
  ca_path = "ca.crt"
  client_cert = ""
  client_cert_key = ""
  client_cert_path = "thing.crt"
  client_priv_key_path = "thing.key"
  mtls = "false"
  priv_key = "thing.key"
  retain = "false"
  skip_tls_ver = "false"
  url = "tcp://mainflux.com:1883"

[[routes]]
  mqtt_topic = "channel/<channel_id>/messages"
  subtopic = "subtopic"
  nats_topic = "export"
  type = "default"
  workers = 10

[[routes]]
  mqtt_topic = "channel/<channel_id>/messages"
  subtopic = "subtopic"
  nats_topic = "channels"
  type = "mfx"
  workers = 10
```
### Http port

- `port` - HTTP port where status of `Export` service can be fetched.
```bash
curl -X GET http://localhost:8170/version
{"service":"export","version":"0.0.1"}%
``` 

### MQTT connection

To establish connection to MQTT broker following settings are needed:
- `username` - Mainflux <thing_id>
- `password` - Mainflux <thing_key>
- `url` - url of MQTT broker

Additionally, you will need MQTT client certificates if you enable mTLS. To obtain certificates `ca.crt`, `thing.crt` and key `thing.key` follow instructions [here](https://mainflux.readthedocs.io/en/latest/authentication/#mutual-tls-authentication-with-x509-certificates).

### Routes 

Routes are being used for specifying which subscriber's topic(subject) goes to which publishing topic.
Currently only MQTT is supported for publishing.
To match Mainflux requirements `mqtt_topic` must contain `channel/<channel_id>/messages`, additional subtopics can be appended.

- `mqtt_topic` - `channel/<channel_id>/messages/<custom_subtopic>`
- `nats_topic` - `Export` service will be subscribed to NATS subject `<nats_topic>.>`
- `subtopic` - messages will be published to MQTT topic `<mqtt_topic>/<subtopic>/<nats_subject>`, where dots in nats_subject are replaced with '/'
- `workers` control number of workers that will be used for message forwarding.
- `type` - specifies message transformation, `default` is for sending messages as they are recieved on NATS with no transformation (so they should be in SenML format for succsefull exporting to Mainflux cloud) and `mfx` is for messages that are being picked up on internal Mainflux NATS bus, messages that are published to local running Mainflux via MQTT will end on NATS, before sending, SenML must be extracted from Mainflux Message and then sent via MQTT to cloud.

Before running `Export` service edit `configs/config.toml` and provide `username`, `password` and `url`
 * `username` - matches `thing_id` in Mainflux cloud instance
 * `password` - matches `thing_key`
 * `channel` - MQTT part of the topic where to publish MQTT data (`channel/<channel_id>/messages` is format of mainflux MQTT topic) and plays a part in authorization.

If Mainflux and Export service are deployed on same gateway `Export` can be confiugred to send messages from Mainflux internal NATS bus to Mainflux in a cloud.
In order for `Export` service to listen on Mainflux NATS deployed on the same machine NATS port must be exposed.
Edit Mainflux [docker-compose.yml][docker-compose]. NATS section must look like below:
```
  nats:
    image: nats:1.3.0
    container_name: mainflux-nats
    restart: on-failure
    networks:
      - mainflux-base-net
    ports:
      - 4222:4222
```
  
## Environment variables

Service will look for `config.toml` first and if not found it will be configured with env variables and new config file specified with `MF_EXPORT_CONFIG_FILE` will be saved with values populated from env vars.  
The service is configured using the [environment variables](env). Note that any unset variables will be replaced with their default values.


For values in environment variables to take effect make sure that there is no `MF_EXPORT_CONFIG_FILE` file.

If you run with environment variables you can create config file:
```bash
MF_EXPORT_PORT=8178 \
MF_EXPORT_LOG_LEVEL=debug \
MF_EXPORT_MQTT_HOST=tcp://localhost:1883 \
MF_EXPORT_MQTT_USERNAME=<thing_id> \
MF_EXPORT_MQTT_PASSWORD=<thing_key> \
MF_EXPORT_MQTT_CHANNEL=<channel_id> \
MF_EXPORT_MQTT_SKIP_TLS=true \
MF_EXPORT_MQTT_MTLS=false \
MF_EXPORT_MQTT_CA=ca.crt \
MF_EXPORT_MQTT_CLIENT_CERT=thing.crt \
MF_EXPORT_MQTT_CLIENT_PK=thing.key \
MF_EXPORT_CONFIG_FILE=export.toml \
../build/mainflux-export&
```
Values from environment variables will be used to populate export.toml

## How to save config via agent

Configuration file for `Export` service can be sent over MQTT using [Agent][agent] service.

```
mosquitto_pub -u <thing_id> -P <thing_key> -t channels/<control_ch_id>/messages/req -h localhost -p 18831  -m  "[{\"bn\":\"1:\", \"n\":\"config\", \"vs\":\"save, export, <config_file_path>, <file_content_base64>\"}]"
```

`vs="save, export, config_file_path, file_content_base64"` - vs determines where to save file and contains file content in base64 encoding payload:
```
b,_ := toml.Marshal(export.Config)
payload := base64.StdEncoding.EncodeToString(b)
```


[conftoml]: (https://github.com/mainflux/export/blob/master/configs/config.toml)
[docker-compose]: (https://github.com/mainflux/mainflux/docker/docker-compose.yml)
[env]: (https://github.com/mainflux/export#environmet-variables)
[agent]: (https://github.com/mainflux/agent)